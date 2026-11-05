---
name: cc-call-apis
description: Use BucCommOmsMashupService to call Call Center mashups from custom code, including paginated lists, and show success or error notifications.
---

# Call mashups from custom code

Call Center code reaches OMS through mashups (`/icc`), not direct API calls. Keep calls
in a service. If the mashup you need doesn't exist, or doesn't return an attribute,
follow [cc-extend-mashups](../cc-extend-mashups/SKILL.md).

## Reuse first

Check the module's shared library before writing a call. `@call-center/<short>-shared`
data services already wrap most mashups:

```
grep -rn "MASHUP_ID" packages/<short>-shared/src/lib --include=*.ts
```

## Single call

```ts
import { Injectable } from '@angular/core';
import { BucCommOmsMashupService } from '@buc/svc-angular';

@Injectable()
export class ExtnOrgService {
  private readonly MASHUP_ID = 'extn_icc.order-details.summary.getOrganizationList';

  constructor(private mashupService: BucCommOmsMashupService) {}

  getOrganizations(input: any): Promise<any> {
    return this.mashupService.callMashup(this.MASHUP_ID, input, {}).toPromise()
      .then(out => this.mashupService.getMashupOutput(out, this.MASHUP_ID));
  }
}
```

The input represents the mashup's `<Input>` element as JSON. The mashup silently
drops attributes its template doesn't accept.

## Paginated list

```ts
// pageAction: 'START' for the first page, then 'NEXT' or 'PREVIOUS'. Pass back the same pageModel.
return this.mashupService.callPaginatedMashup(this.MASHUP_ID, input, pageAction, pageSize, pageModel, {})
  .toPromise()
  .then(out => this.mashupService.getPaginatedMashupOutput(out, this.MASHUP_ID));
```

The mashup must use `ICCBasePaginatedMashup`.

## Notify the user

```ts
import { BucNotificationModel, BucNotificationService } from '@buc/common-components';

this.notificationService.send([
  new BucNotificationModel({ statusType: 'success', statusContent: this.translate.instant('CUSTOM.SAVED') })
]);
```

`statusType` is `success`, `error`, `info`, or `action`. Keep messages as translation
keys in `src-custom/assets/custom/i18n/en.json`. Always handle errors so a failed call
doesn't leave a spinner running.

## Verify

Ask the user to run the flow and check the `/icc` request in the network tab: the
mashup ID, the input, the status, and the response. If the response lacks the
attribute, extend the mashup template. The code doesn't need to change.
