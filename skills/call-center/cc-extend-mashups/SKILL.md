---
name: cc-extend-mashups
description: Get new data into the Call Center UI by extending the /icc mashup layer. Covers incremental, override, and new mashups, custom Java mashup classes, naming rules, and how to build and deploy them. Needed whenever a new column, field, or search attribute is blank.
---

# Extend mashups

Every UI call goes to the mashup server (`/icc`), which calls OMS APIs with fixed
input and output templates. If an attribute isn't in a template, the UI can't see
or send it. Most column, field, and search customizations need a mashup change.

Find the mashup ID a page uses in the browser's network tab (the request to `/icc`).
The shipped XML files are in `<INSTALL_DIR>/repository/eardata/icc/war/mashupxmls/`.

## Choose the extension type

| Need | Type | File suffix | Upgrade impact |
|---|---|---|---|
| Add attributes to an existing mashup's input or output | Incremental | `_incrementalmashups.xml` | Low |
| Replace a template wholesale, swap the Java class, or call a service | Override | `_overridemashups.xml` | You own the whole mashup |
| New data for your own component | New mashup | `_mashups.xml` | None |

Put all files in `<runtime>/extensions/icc/webpages/mashupxmls/`. You can use
subfolders, for example `order/order-summary/`.

## Incremental

Copy the original mashup and add only the new attributes:

```xml
<mashups>
  <mashup id="icc.common-components.getOrganizationHierarchy" description="..."
          mashuptype="XAPI" transactional="true" skipDataProvider="true">
    <classInformation name="com.ibm.icc.common.mashups.ICCBaseMashup"/>
    <API Name="getOrganizationHierarchy">
      <Input>
        <Organization DisplayLocalizedFieldInLocale="xml:CurrentUser:/User/@Localecode" IsEnterprise="" OrganizationCode=""/>
      </Input>
      <Template>
        <Organization InventoryOrganizationCode="" OrganizationCode="" OrganizationName=""
                      ParentOrganizationCode="" PrimaryEnterpriseKey="" OrganizationKey=""/>  <!-- added OrganizationKey -->
      </Template>
    </API>
  </mashup>
</mashups>
```

For a search field, add the attribute to the search mashup's `<Input>`. For example:
`icc.order.order-search-results.getPaginatedOrderList`:

```xml
<API Name="getOrderList"><Input>
  <Order>
    <OrderLine KitCode=""><CustomAttributes BUC_IsPreorder=""/><Item ItemID=""/></OrderLine>
    <PaymentMethod DisplayCreditCardNo=""/>
  </Order>
</Input></API>
```

## Override

Copy the full mashup into `*_overridemashups.xml` and change it. Use this for a new
Java class:

```xml
<mashup id="icc.order.summary.getCompleteOrderDetails" ...>
  <classInformation name="com.mycompany.cc.ExtnGetCompleteOrderDetails"/>
  ...
</mashup>
```

Or to call a custom service instead of an API:

```xml
<mashup id="icc.order.summary.getCompleteOrderLineList" mashuptype="XAPI" transactional="true" skipDataProvider="false">
  <classInformation name="com.ibm.icc.common.mashups.ICCBasePaginatedMashup"/>
  <API FlowName="ExtnGetCompleteOrderLineListService">
    <Input>...</Input>
    <Template>...</Template>
  </API>
</mashup>
```

The service's API template must include every attribute the UI needs.

## New mashup

Name it `extn_icc.<module>.<component>.<task>`:

```xml
<mashups>
  <mashup id="extn_icc.order-details.summary.getOrganizationList" description="Get the organization's customers"
          mashuptype="XAPI" transactional="true" skipDataProvider="true">
    <classInformation name="com.ibm.icc.common.mashups.ICCBaseMashup"/>
    <API Name="getOrganizationList">
      <Input><Organization CustomerMasterOrganizationCode="" DisplayLocalizedFieldInLocale=""/></Input>
      <Template>
        <OrganizationList>
          <Organization OrganizationCode="" OrganizationName=""><BillingPersonInfo City="" Country="" ZipCode=""/></Organization>
        </OrganizationList>
      </Template>
    </API>
    <AlternateResourceIds><AlternateResourceId altResourceId="ICCSYS00001"/></AlternateResourceIds>
  </mashup>
</mashups>
```

| Item | Notes |
|---|---|
| `classInformation` | `ICCBaseMashup`, `ICCBasePaginatedMashup` (paginated lists), or your subclass |
| `transactional` | `true` for XAPI calls |
| `resourceId` / `AlternateResourceIds` | Authorization. With neither, everyone can call it. Alternates are used only when `resourceId` is absent |
| `DisplayLocalizedFieldInLocale` | Pass it in new mashups so localized values come back (`fr_FR`, `en_US`, ...) |
| `APINamespace`, `mashupRef` | For multi-API mashups |

Call it from the UI by ID, the same way JSON `fetch` blocks do (`"api": "<id>", "type": "mashup"`).

## Custom Java class

1. Create a Java project. Add jars from `<PATH_TO_EAR>/smcfs.ear/lib` and
   `smcfs.ear/icc.war/WEB-INF/lib` to the build path.
2. Extend `ICCBaseMashup` and override `massageOutput` (or `massageInput`):

   ```java
   package com.mycompany.cc;

   public class ExtnGetCompleteOrderDetails extends ICCBaseMashup {
     @Override
     public Element massageOutput(Element outEl, SCUIMashupMetaData md, SCUIContext ctx) {
       outEl.setAttribute("ExtnFlag", "Y");
       return outEl;
     }
   }
   ```

3. `jar cf custom_mashups.jar ./*` from the output folder. Copy it to
   `<runtime>/repository/eardata/icc/extn/WEB-INF/lib/` (create the folders).
4. Reference it from an override mashup.

## Build and deploy

| Environment | Build | Deploy |
|---|---|---|
| DTK | `./sci_ant.sh -f ../devtoolkit/devtoolkit_extensions.xml` | `./om-compose.sh update-extn <full path to extensions.jar>` |
| Containers | The mashup WAR (`icc.war`) is part of the OMS EAR, so build a custom OM image | Deploy through the operator |
| On premises | Rebuild the EAR | Redeploy and restart the app server |

Mashup changes go in the OMS extensions, not in the UI customization package.

## Verify

After deploying the extensions, ask the user to reload the page and check the `/icc`
request in the network tab: the response should now carry the new attribute.

- **Mashup metadata not found:** wrong `id`, the file isn't in `mashupxmls/`, or it
  lacks a recognized suffix.
- **Not extensible:** the shipped mashup has `extensible="false"`.
- **No data, no error:** the user lacks the mashup's resource.
