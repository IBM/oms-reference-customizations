---
name: buc-call-apis
description: Call OMS APIs and service-domain REST APIs from custom Order Hub code, handle actions, and show success or error notifications.
---

# Call APIs from custom code

Keep API calls in a service rather than a component. Generate one with `ng g s <name>` in a
`services` folder next to your feature.

## OMS APIs

`BucCommOmsRestAPIService` invokes any OMS API by name:

```ts
import { Injectable } from '@angular/core';
import { BucCommOmsRestAPIService } from '@buc/svc-angular';

@Injectable({ providedIn: 'root' })
export class MyDataService {
  constructor(private omsApi: BucCommOmsRestAPIService) {}

  getReasonCodes(enterpriseCode: string, docType: string) {
    return this.omsApi.invokeOMSRESTApi('getCommonCodeList', {
      CallingOrganizationCode: enterpriseCode,
      CodeType: 'NOTES_REASON',
      DocumentType: docType
    }, {});
  }

  saveOrder(order: any) {
    return this.omsApi.invokeOMSRESTApi('modifyFulfillmentOptions', order, {});
  }
}
```

The input object is the API's input expressed as JSON. Check the Javadoc for the
exact attribute names. An incorrect name is silently ignored rather than rejected.

For an OMS **service** (a flow, not an API), use `invokeOMSCustomService` on the same class.
Its fourth argument holds optional extra request headers:

```ts
this.omsApi.invokeOMSCustomService('RequestForReassignOrderRelease', input, null, customHeaders);
```

## Service-domain REST APIs

For REST APIs outside OMS (inventory, for example), use
`BucCommBEHttpWrapperService`, which resolves the domain's base path and auth
headers:

```ts
import { Injectable } from '@angular/core';
import { BucCommBEHttpWrapperService } from '@buc/svc-angular';
import { Observable, throwError } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ReservationService {
  private resourceDomain = 'inventory';
  private domain: any;
  private options: any;

  constructor(private http: BucCommBEHttpWrapperService) {
    this.domain = BucCommBEHttpWrapperService.getPathPrefix(this.resourceDomain);
    this.options = BucCommBEHttpWrapperService.getRequestOptions(this.resourceDomain);
  }

  createReservation(tenantId: string, input: any): Observable<any> {
    if (!tenantId) {
      return throwError(new Error('Missing required parameter: tenantId'));
    }
    const path = '/{tenant}/v1/reservations'.replace('{tenant}', tenantId);
    return this.http.post(this.domain + path, this.resourceDomain, null, input, this.options);
  }

  getReservation(tenantId: string, referenceId: string): Observable<any> {
    if (!tenantId) {
      return throwError(new Error('Missing required parameter: tenantId'));
    }
    const path = '/{tenant}/v1/reservations?reference={referenceId}'
      .replace('{tenant}', tenantId)
      .replace('{referenceId}', referenceId);
    return this.http.get(this.domain + path, this.resourceDomain, null, this.options);
  }
}
```

Guard required path parameters like `tenantId` before building the URL — an empty
value produces a malformed path and a confusing 404.

The last argument (`this.options`) is optional headers. Omit it unless the domain needs
custom headers.

If one method needs a different domain from the one its class uses, resolve that domain
inside the method. Leave the class fields unchanged:

```ts
// class domain is 'inventory_buc'; this endpoint lives under 'inventory'
getSupplyTypes(tenantId: string) {
  const resDomain = 'inventory';
  const domain = BucCommBEHttpWrapperService.getPathPrefix(resDomain);
  return this.http.get(`${domain}/${tenantId}/v1/configuration/supply_types`, resDomain);
}
```

The tenant ID comes from app context, not from your own config:

```ts
import { BucSvcAngularStaticAppInfoFacadeUtil } from '@buc/svc-angular';

this.tenantId = BucSvcAngularStaticAppInfoFacadeUtil.getInventoryTenantId();
```

## Reuse what already exists

Before writing a call, check the module's shared library — most common calls are
already there and handle paging and error shapes for you:

```ts
import { InventoryAvailabilityService, InventoryContextService } from '@buc/inventory-shared';
```

A module's context service holds the search criteria and selections the user
arrived with. Read from it rather than threading parameters through route state.

## Combine calls

Batch several OMS APIs into one request with `multiApi`:

```ts
const API = personInfoKeys.map(key => ({
  Name: 'getPersonInfoList',
  Input: { PersonInfo: { PersonInfoKey: key } }
}));

this.omsApi.invokeOMSRESTApi('multiApi', { API }, {}).subscribe((res: any) => {
  const results = Array.isArray(res.API) ? res.API : [res.API];
  results.forEach(r => /* r.Output is one API's response */);
});
```

A single call comes back as an object, not an array, so normalize before iterating.

Use `forkJoin` only for calls that can't be batched, such as an OMS call alongside a
service-domain REST call:

```ts
forkJoin([this.getReservation(), this.getAvailability()]).pipe(
  map(([reservations, availability]: any[]) => /* merge into rows */),
  catchError(() => { this.multiModel.totalDataLength = 0; return of([]); })
);
```

Always use `catchError`. Without it, a failed call leaves the table loading indefinitely.

## Paginated lists

Fetch list data through the module's shared `CommonService`, not
`invokeOMSRESTApi('getPage', ...)`. The wrapper tracks the `PageSetToken`
between pages:

```ts
import { CommonService, Templates } from '@buc/order-shared';

CommonService.getPage(input, Templates.OrderList, pageNumber, pageSize, refresh);
CommonService.getNextPage(input, Templates.OrderList, pageSize, previousPage, refresh);
```

`getNextPage` uses `NEXTPAGE` pagination: pass the last record of the previous page, or
`null` on the first load. `refresh: false` reuses the stored token. The template is an
`{ api, id }` reference. To add attributes to it, see
[buc-configure-getpage-templates](../buc-configure-getpage-templates/SKILL.md).

## Handle an action

Actions are dispatched by name. Subscribe in a service to react to one:

```ts
import { ActionProcessorService } from '@buc/common-components';

constructor(public actionSvc: ActionProcessorService) {
  actionSvc.select<ActionParams>(CustomConstants.ACTION_IDS_MY_ACTION).pipe(
    map(res => this.execute(res.params))
  ).subscribe();
}

async execute(params: ActionParams) {
  this.myState = params.data?.item;     // the selected row
}
```

Register through `AppCustomizationImpl.providers`, preserving existing entries:

```ts
MyActionService,
{ provide: CUSTOM_FEATURE_ACTIONS,
  useValue: [{ name: CustomConstants.ACTION_IDS_MY_ACTION, action: MyActionService }],
  multi: true }
```

**Mandatory:** list the service class itself in `providers` as well as the token
mapping, or the handler is never created. To choose between `CUSTOM_FEATURE_ACTIONS`
and `CUSTOM_ACTIONS`, follow step 5 of
[buc-add-custom-action](../buc-add-custom-action/SKILL.md).

## Notify the user

```ts
import { BucNotificationModel, BucNotificationService } from '@buc/common-components';

showNotification(statusType, message) {
  this.bucNotificationService.send([
    new BucNotificationModel({ statusType, statusContent: message })
  ]);
}
```

Keep the messages as translation keys in `src-custom/assets/custom/i18n/en.json`,
for example `custom.SUCCESS_RESERVATION` and `custom.ERROR_RESERVATION`.

## Verify

Ask the user to run the flow and check the **Network** tab: the request payload,
the status, and the response body. Ask them to paste the response if the data
doesn't appear where you expect. The mapping is usually the problem, not the call.

For route customization, first complete [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).
