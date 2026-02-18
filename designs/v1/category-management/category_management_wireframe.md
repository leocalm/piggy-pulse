# CATEGORY_MANAGEMENT_ASCII_WIREFRAME.md

## Desktop

--------------------------------------------------------------------------------
Categories                                                [+ Add Category]
Define and organize your budgeting structure
[ Search categories... ________________________________ ]
--------------------------------------------------------------------------------

INCOMING
--------------------------------------------------------------------------------
Salary                                   Active     [Edit] [Archive]
Bonus                                    Active     [Edit] [Archive]
Refund                                   Active     [Edit] [Archive]

OUTGOING
--------------------------------------------------------------------------------
Dining                                   Active     [Edit] [Archive] [+ Subcategory]
    Restaurants                          Active     [Edit] [Archive]
    Coffee                               Active     [Edit] [Archive]

Housing                                  Active     [Edit] [Archive] [+ Subcategory]
    Rent                                 Active     [Edit] [Archive]
    Utilities                            Active     [Edit] [Archive]

Groceries                                Active     [Edit] [Archive]
Transport                                Active     [Edit] [Archive]

ARCHIVED (collapsed)
--------------------------------------------------------------------------------
[▸] Archived (3)
    Old Category                         Archived   [Edit] [Restore]
    Old Parent                           Archived   [Edit] [Restore]
        Old Child                        Archived   [Edit] [Restore]


================================================================================
Create Category (Modal)

Title: Create Category
Fields:
[ Name: ________________________________ ]
[ Type: (Incoming v) ]
[ Icon: (optional) ____ ]
[ Description: _________________________ ]
[ Cancel ] [ Create ]


================================================================================
Create Subcategory (Modal)

Title: Create Subcategory
[ Parent: Dining (frozen) ]
[ Type: Outgoing (frozen) ]
[ Name: ________________________________ ]
[ Icon: (optional) ____ ]
[ Description: _________________________ ]
[ Cancel ] [ Create ]


================================================================================
Edit Category (Modal)

Title: Edit Category
[ Name: ________________________________ ]
[ Type: Outgoing (frozen if tx > 0) ]
[ Parent: (hidden in v1) or (frozen if tx > 0) ]
[ Icon: (optional) ____ ]
[ Description: _________________________ ]

Helper (if frozen):
- Type cannot be changed after the first transaction exists.
- Parent cannot be changed once transactions exist.

[ Cancel ] [ Save ]


================================================================================
Archive Confirmation

"This will archive the category. Historical transactions remain unchanged.
Archived categories cannot be used for new transactions."

[ Cancel ] [ Archive ]


Blocking variant (parent has active children):
"This category has active subcategories. Archive subcategories first."
[ OK ]


================================================================================
Delete Confirmation (only if allowed)

"This category has no transactions and no subcategories.
It will be permanently removed."

[ Cancel ] [ Delete ]


## Mobile (Cards + Drawer)

--------------------------------------------------------------------------------
Categories                                         [+]
[ Search... __________________ ]
--------------------------------------------------------------------------------

INCOMING
[ 💵 Salary ]
Active
[ Edit ] [ Archive ]

OUTGOING
[ 🍽 Dining ]
Active
[ Edit ] [ Archive ] [ + Subcategory ]
    [ Restaurants ] Active  [ Edit ] [ Archive ]
    [ Coffee ]      Active  [ Edit ] [ Archive ]

ARCHIVED
[▸] Archived (3)

Create/Edit uses bottom drawer. Actions are always visible (no hover).
