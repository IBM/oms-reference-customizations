---
name: cc-customize-payment-methods
description: Build the capture UI for Call Center payment types (credit card, debit card, SVC, or your own) as an iframe app in call-center-custom, wire it with paymentTypes in app-bootstrap-config.json, and implement the postMessage contract.
---

# Custom payment method UI

Call Center has no built-in capture screen for payment types like `CREDIT_CARD`,
`DEBIT_CARD`, or `SVC`. When a user picks one, the base UI loads your page in an
iframe (`buc-cc-iframe-loader`), either on the page or in a modal. Your page captures the
details and posts them back. This works on every page that takes payments.

`call-center-custom` is a plain Angular app with a working sample:

```
call-center-custom/src/app/features/payments/custom-payment-method/custom-payment-method.component.ts
route: payments/customer-payment-method/:paymentType
```

Use this sample as your starting point.

## 1. Map payment types to URLs

`callcenter-code/shell-ui/assets/app-bootstrap-config.json`:

```json
{
  "paymentTypes": [
    { "id": "CREDIT_CARD", "frameUrl": "/call-center-customization/call-center-custom/payments/customer-payment-method/CREDIT_CARD" },
    { "id": "DEBIT_CARD",  "frameUrl": "/call-center-customization/call-center-custom/payments/customer-payment-method/DEBIT_CARD" },
    { "id": "SVC",         "frameUrl": "/call-center-customization/call-center-custom/payments/customer-payment-method/SVC" }
  ]
}
```

`id` is the payment type ID configured in OMS.

| URL | When |
|---|---|
| Relative (recommended) | Your UI is built into the customization package. It's same-site, so it reuses the parent's session and can call mashups with the `csrfToken` |
| Absolute | Reusing an existing UI on another domain (for example your web store). It's cross-site: no session, no mashup calls. It can only capture data and post it back |

## 2. Implement the message contract

All messages use the shape `{ action: "<NAME>", data: <payload> }`.

**Parent → your frame**

| Action | Payload |
|---|---|
| `RENDER_VIEW_DATA` | Sent after load: `csrfToken`, `language`, `locale`, `timeZone`, `userId`, `userLocale`, `mashupURL`, `resourcesForUser`, `accessibleOrgListForUser`, `devMode` |
| `IFRAME_ONLOAD_COMPLETE` | No payload |
| `RENDER_VIEW_CONTEXT` | Page context (below) |

`RENDER_VIEW_CONTEXT` for **customer payment methods**:
`customer`, `customerContact`, `currencyList`, `organizationCode`, `paymentType`,
`mode` (`CREATE` or `MODIFY`), and `modifyPaymentMethodInput` (`paymentType`,
`paymentMethod`).

For **order and return pages**: `order`, `paymentType`, `amountDue` (the refund
amount for returns), `existingPaymentMethods`.

**Your frame → parent**

| Action | Payload |
|---|---|
| `FRAME_SCREEN_UPDATED` | Send whenever your layout changes, so the parent resizes the frame |
| `CUSTOM_PAYMENT_METHOD` | On every change, valid or not |

`CUSTOM_PAYMENT_METHOD` payload:

```js
// customer payment method
{ isValid, PaymentMethod: isValid ? customerPaymentMethod : {} }
// order and return pages
{ isValid, PaymentMethod: isValid ? paymentMethod : {}, saveToProfile, CustomerPaymentMethod /* required if saveToProfile */ }
```

Sketch:

```ts
@HostListener('window:message', ['$event'])
onMessage({ data }: MessageEvent) {
  if (data?.action === 'RENDER_VIEW_DATA')    { this.session = data.data; }
  if (data?.action === 'RENDER_VIEW_CONTEXT') { this.context = data.data; this.initForm(); }
}

private emit() {
  window.parent.postMessage({ action: 'CUSTOM_PAYMENT_METHOD',
    data: { isValid: this.form.valid, PaymentMethod: this.form.valid ? this.toPaymentMethod() : {} } });
  window.parent.postMessage({ action: 'FRAME_SCREEN_UPDATED' });
}
```

Never put raw card numbers in the payment method. Tokenize with your payment
provider inside the frame and send only the token and display fields.

## 3. Run locally in the DTK

1. In `devtoolkit_docker/compose-callcenter/docker/cc-docker-compose.properties`:

   | Property | Value |
   |---|---|
   | `ENABLE_CUSTOM_MODULE_DEV_MODE` | `true` (proxy from the base container to your dev server) |
   | `CALL_CENTER_CUSTOM_SERVER` | dev server host (default `localhost`) |
   | `CALL_CENTER_CUSTOM_PORT` | `9100` |
   | `PAYMENT_TYPES_CONFIG_BASE64` | base64 of the `paymentTypes` array |

   ```
   echo -n '[{"id":"CREDIT_CARD","frameUrl":"/call-center-customization/call-center-custom/payments/customer-payment-method/CREDIT_CARD"}]' | base64 -w0
   ```

2. Apply with `./cc-compose.sh setup`.
3. `cd call-center-custom && yarn start` (port 9100).
4. In Call Center, add a payment method on a customer, or pay on create order, and
   pick your type.

## 4. Deploy

1. Keep `paymentTypes` in `shell-ui/assets/app-bootstrap-config.json`.
2. In `call-center-custom/package.json`:
   `"package-customization": "yarn cc-components:prod && yarn build"`.
3. Include `call-center-custom` in the build:

   ```
   ./build-customization.sh build-ui call-center-order,call-center-custom
   ./build-customization.sh package-jar call-center-order,call-center-custom
   ```

`call-center-custom` isn't part of the base UI. It only ships through your
customization package.

## Verify

Ask the user to add the payment type to a customer and at a create-order payment
step. Confirm that the frame renders and resizes, that **Save** or **Next**
becomes enabled only when the form is valid, and that the saved payment method has
the expected fields.

If the frame never renders, check `frameUrl` and the dev proxy properties. If it renders but
**Save** or **Next** never becomes enabled, either `CUSTOM_PAYMENT_METHOD` isn't posted or
`isValid` stays false.
