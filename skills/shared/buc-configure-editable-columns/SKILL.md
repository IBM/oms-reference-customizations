---
name: buc-configure-editable-columns
description: Make columns in an existing Order Hub/Call Center table editable by registering an editable-fields service against the table configuration.
---

# Add editable columns to a table

Editable columns need a service token in the table config and a service that
implements `EditableFieldsInterface`. The service handles edits and writes the
results back into the table's data.

Once registered, the `isEditable`, `fieldConfig`, and `options` header attributes
become available on that table's columns.

**Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## 1. Add the token to the table config

In your `buc-table-config.json`, declare `tokenId` alongside `headers`:

```json
{
  "order-table": {
    "name": "order-table",
    "aliases": ["inbound-order-table"],
    "tokenId": "abctable-custom-service",
    "headers": [ ]
  }
}
```

`aliases` applies the same configuration to related tables — here, the inbound
order search results table gets the same columns as the outbound one.

To find the table's schema name, ask the user to open the page with the browser
**Console** open and read the last `BaseTableComponent.initializeTable` message.

## 2. Write the service

```ts
import { Injectable } from '@angular/core';
import { BucTableHeaderItem, EditableFieldsInterface } from '@buc/common-components';
import { OrderlineListTableComponent } from '../../app/features/order/orderline-list-table/orderline-list-table.component';

@Injectable()
export class ABCTableCustomEditableFieldsService implements EditableFieldsInterface {
  tableContext: OrderlineListTableComponent;

  /** Required. Keeps a handle on the table instance. */
  initializeTableParentContext(parentContext) {
    this.tableContext = parentContext;
  }

  onDropdownSelection(event, data) {
    const key = data.item.key;
    this.tableContext.allTableData.forEach((row) => {
      if (row.key === key && data.headerId === 'lineTypeCustom') {
        row.item.lineType = data.value;
      }
    });
    this.tableContext.tableDataChange.emit(this.tableContext.allTableData);
  }
}
```

`initializeTableParentContext` is required to access the table's data.

The interface declares eleven optional hooks, implement only the ones you need:

| Method | Use |
|---|---|
| `processDefaultValueInField(header, data)` | Push a default set in the JSON config back into the parent context |
| `getDataForField(header, data)` | Return custom data for a field, for example a filtered dropdown list |

The other hooks are `isFieldDisabled`, `onDropdownSelection`,
`onComboboxSelection`, `onNumberChange`, `onDateChange`, `onRadioChange`,
`inputValueChange`, `onToggleChange`, and `onLinkClick`. Check their signatures
in the installed `EditableFieldsInterface` before implementing them.

After changing row data, always emit `tableDataChange` with the full data set.
Mutating rows without emitting leaves the table showing stale values.

## 3. Register it

In `src-custom/app/app-customization.impl.ts`, provide the service under the same
token string:

```ts
import { ABCTableCustomEditableFieldsService } from './services/abctable-custom.service';

export class AppCustomizationImpl {
  static readonly components = [];
  static readonly providers = [
    { provide: 'abctable-custom-service', useClass: ABCTableCustomEditableFieldsService }
  ];
  static readonly imports = [];
}
```

Use `useClass` rather than `useValue`. The `provide` string must match
`tokenId` exactly.

## 4. Configure the editable columns

With the service registered, add `isEditable`, `fieldConfig`, and `options` to the
column definitions in `buc-table-config.json` to control which columns are editable
and what control each one uses.

## 5. Verify

Ask the user to reload the frame, open the table, edit a cell, and confirm the new
value persists after the edit and after sorting again. A value that reverts means
`tableDataChange` wasn't emitted.
