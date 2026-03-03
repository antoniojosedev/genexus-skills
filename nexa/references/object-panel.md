---
name: object-panel
description: Screen definition for Android, Apple, Angular, or Web environments
---

Defines user interface screens for multiple platforms with layout, events, and data binding

---

# DEFINITION
A `Panel` object defines a screen for Android, Apple, Angular, or Web environments

Types: `Panel` (or `SDPanel`), `WebPanel`, `MasterPanel`, `MasterPage`, `WebComponent`, `Stencil`

---

# SYNTAX
~~~
<type> <name>
{
	#Events
		<events>
	#End

	#Rules
		<rules>
	#End

	#Conditions
		<conditions>
	#End

	#Variables
		<variables>
	#End

	#Layout
		<layout>
	#End

	#Properties
		<properties>
	#End

	#Documentation
		<documentation>
	#End
}
~~~

Where:
- `<type>`: Object type: `SDPanel` (default), `WebPanel`, `WebComponent`, `MasterPage`, `MasterPanel`, `Stencil`
- `<name>`: Object name using alphanumeric or underscore, starting with letter
- `<events>`: Event handlers triggered by user actions
- `<rules>`: Business rules (parm, etc.)
- `<conditions>`: Boolean filters (comparisons, operators, functions, formulas); multiple lines imply `AND`
- `<variables>`: Variable definitions with `DataType`
- `<layout>`: GXML layout definition (XML-based) for structure and control composition
- `<properties>`: Optional object properties in TOML syntax
- `<documentation>`: Optional object documentation (check [common-markdown](./common-markdown.md))


---

# OUTPUT
Use [global-output](./global-output.md) with `<type>` value:
- For `Panel` object → `panel`
- For `WebPanel` object → `webpanel`
- For `Stencil` object → `stencil`
- For `WebComponent` object → `webcomponent`
- For `MasterPanel` object → `masterpanel`
- For `MasterPage` object → `masterpage`

---

# CONSTRAINTS
- Use [global-constraints](./global-constraints.md)
- Include [common-standard-variables](./common-standard-variables.md) according to panel context
- Place code only inside `Panel` object sections
- Allow only `#Layout`, `#Variables`, `#Properties`, `#Documentation` in `Stencil` object
- Events use qualifiers when needed: `[WEB]`, `[WIN]`, `[TEXT]`

---

# CONVENTIONS
- Define screen purpose, primary actions, and navigation path before `#Layout`
- Define UI states per screen: `Default`, `Loading`, `Empty`, `Error`, `Success`
- Define interaction trigger and feedback for each actionable control
- Define responsive adaptations for mobile, tablet, and desktop
- Define layout structure in `Panel` object; define visual styling in `DesignSystem` object
- Reference only pre-defined style classes from `DesignSystem` object
- Set `Style` property to target `DesignSystem` object when style contract applies
- Use semantic class names in layout; avoid visual names

---

# EVENTS
See [common-events](./common-events.md)

Allowed event names:
- `Start`
- `ClientStart` (non-web)
- `[<grid-name>.]Refresh`
- `[<grid-name>.]Load`
- `Enter` (web only)
- `Back` (non-web only)
- `'<custom-name>'` (for buttons)
- `<control-name>.<event-name>`

Common control event names:
- Web: `Click`, `DblClick`
- Native: `Tap`, `DoubleTap`, `LongTap`, `Swipe`, `SwipeTop`, `SwipeLeft`, `SwipeRight`, `SwipeBottom`

Execution order:
1. `Start`
2. `Refresh` and then each `<grid>.Refresh`
3. `Load` after `Refresh`, and `<grid>.Load` after each `<grid>.Refresh` (one execution per loaded row)
4. User interaction events: `Enter`, `Back`, `'<custom-name>'`, `<control-name>.<event-name>`

Note:
- `Load` can refer to event name or command name depending on context

Example:
~~~
Event 'Calculate'
	&Total = &Price * &Quantity
	msg(format(!"Total: %1", &Total), status)
EndEvent
~~~

---

# LAYOUT
Declarative layout using XML-based format called GXML with:
- `<smart>`: Modern div-based responsive layout
- `<table>`: Legacy table-based layout
- `<canvas>`: Absolute positioning with overlapping
- `<grid>`: Data grid with items
- `<input>`, `<label>`, `<button>`, `<image>`: UI controls
- `<stencil>`: Embedded reusable layout

Scope:
- Define structure and control composition in `#Layout` section
- Define visual styling in `DesignSystem` object classes

Size units:
- `px` (Web), `dip` (native) for absolute
- `%` for relative
- Other units forbidden

Layout constraints:
- For `canvas`, each direct child must define:
	* Both `width` and `height`
	* At least one horizontal anchor (`left` or `right`)
	* At least one vertical anchor (`top` or `bottom`)
- For `smart` and `table`, `columnsStyle` and `rowsStyle` must not overflow or underflow container size
- For `stencil`, each mapped child must define `controlNameForStencil`
- For `image`, use `source` for static image objects and `attribute` for bound data

---

# EXAMPLES

## Example 1
Simple Panel with Grid
~~~
Panel CustomerList
{
	#Events
		Event Start
			&Customers.Clear()
		EndEvent

		Event Load
			For &Customer in ListCustomersDP()
				&Customers.Add(&Customer)
				load
			EndFor
		EndEvent

		Event GridCustomers.ItemClick
			ViewCustomer(&Customer.CustomerId)
		EndEvent
	#End

	#Rules
	#End

	#Conditions
	#End

	#Variables
		Customers [ DataType = 'CustomerInfo', Collection = 'True' ]
		Customer [ DataType = 'CustomerInfo' ]
	#End

	#Layout
		<layout>
			<view>
				<smart name="MainTable" width="100%" height="100%" columnsStyle="100%" rowsStyle="60dip;100%">
					<row>
						<cell hAlign="Center" vAlign="Middle">
							<label name="Title" caption="Customers" class="screen-title" />
						</cell>
					</row>
					<row>
						<cell>
							<grid name="GridCustomers" controlType="smart" autoGrow="True">
								<smart name="GridCustomerItem" width="100%" height="80dip">
									<row>
										<cell>
											<input attribute="&amp;Customer.CustomerName" readonly="True" />
										</cell>
									</row>
								</smart>
							</grid>
						</cell>
					</row>
				</smart>
			</view>
		</layout>
	#End

	#Properties
		"Captiin" = "Products List"
		"Style" = "MyDesignSystem"
	#End
}
~~~

## Example 2
Panel with Events
~~~
Panel ProductDetail
{
	#Events
		Event Start
			&ProductId = &Parm.ProductId
		EndEvent

		Event Refresh
			For Each Product
				Where ProductId = &ProductId
				&ProductName = ProductName
				&ProductPrice = ProductPrice
				&ProductImage = ProductImage
			EndFor
		EndEvent

		Event 'BuyNow'
			AddToCart(&ProductId)
			msg(!"Added to cart", status)
		EndEvent
	#End

	#Rules
		parm(in: &Parm);
	#End

	#Conditions
	#End

	#Variables
		Parm [ DataType = 'ProductDetailParm' ]
		ProductId [ DataType = 'Numeric(10.0)' ]
		ProductName [ DataType = 'VarChar(128)' ]
		ProductPrice [ DataType = 'Numeric(10.2)' ]
		ProductImage [ DataType = 'Image' ]
	#End

	#Layout
		<layout>
			<view>
				<smart name="Container" width="100%" height="100%" columnsStyle="100%" rowsStyle="200dip;60dip;60dip;100%">
					<row>
						<cell hAlign="Center">
							<image attribute="&amp;ProductImage" width="100%" height="200dip" />
						</cell>
					</row>
					<row>
						<cell>
							<input attribute="&amp;ProductName" readonly="True" class="product-name" />
						</cell>
					</row>
					<row>
						<cell>
							<input attribute="&amp;ProductPrice" readonly="True" class="product-price" />
						</cell>
					</row>
					<row>
						<cell hAlign="Center" vAlign="Middle">
							<button name="BtnBuy" caption="Buy Now" event="'BuyNow'" class="action-button" />
						</cell>
					</row>
				</smart>
			</view>
		</layout>
	#End

	#Properties
		"Caption" = "Product Detail"
		"Style" = "MyDesignSystem"
	#End
}
~~~
