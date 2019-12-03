\_\_TOC\_\_

# Overview

There are 2 parts of information stored in cells, one is "data" and
another is '"style"'. In this section, we are going to introduce the
"style" part which includes alignment, border, border color, font
family, font size, and font style.

Spreadsheet supported border style and font depend upon a browser's
capability.

| Style Feature | Limitation                                                                                           |
| ------------- | ---------------------------------------------------------------------------------------------------- |
| Font Family   | Because of browser limitation, available fonts depend on installed fonts on client side              |
| Border Style  | Because of browser limitation, only **solid**/**dashed**/ **dotted** border style are supported now. |

To get "style" information stored in
<javadoc directory="keikai">io.keikai.api.model.CellStyle</javadoc>
object, you must get
<javadoc directory="keikai">io.keikai.api.Range</javadoc> object first.
Then, we can get a cell's alignment, border setting, and cell color via
`CellStyle`. Every getter method of `CellStyle` has a clear name to
indicate what information it returns. Please refer its javadoc for
complete list. We just introduce some of them for explanation.

# Alignment

``` java
// get Range object for a cell 
Range range = Ranges.range(spreadsheet.getSelectedSheet(), rowIndex, columnIndex);
// get CellStyle
CellStyle style = range.getCellStyle();

//horizontal alignment
Alignment alignment = style.getAlignment();
//vertical alignment
VerticalAlignment verticalAlignment = style.getVerticalAlignment();
```

# Border

``` java
// get Range object for a cell 
Range range = Ranges.range(spreadsheet.getSelectedSheet(), rowIndex, columnIndex);
// get CellStyle
CellStyle style = range.getCellStyle();

//border type
BorderType borderType = style.getBorderTop();

//color
Color color = style.getBorderTopColor();
```

There is one corresponding method to get its border and border color
respectively for each side (top, bottom, left, and right) of a cell.

# Cell Background Color

``` java
// get Range object for a cell 
Range range = Ranges.range(spreadsheet.getSelectedSheet(), rowIndex, columnIndex);
// get CellStyle
CellStyle style = range.getCellStyle();

String colorCode = style.getBackgroundColor().getHtmlColor();
```

# Font

Those information about font can be retrieve via
<javadoc directory="keikai">org.zkoss.api.model.Font</javadoc>, and we can
get this object by `CellStyle`'s `getFont()`. Here are some examples:

``` java
// get Range object for a cell 
Range range = Ranges.range(spreadsheet.getSelectedSheet(), rowIndex, columnIndex);
// get CellStyle
Font font = range.getCellStyle().getFont();

//font family name, e.g. Arial
font.getFontName();

//font size, e.g. 12, 14
font.getFontHeightInPoint()

font.getColor();

//could return Boldweight.BOLD or Boldweight.NORMAL 
font.getBoldweight();

font.isItalic();
font.isStrikeout();

//return Font.Underline
font.getUnderline();
```

# Change Style

In order to save you from complicated underlying implementation, we
provide a utility class
<javadoc directory="keikai">io.keikai.api.CellOperationUtil</javadoc>
to change a cell range's style and it supports almost all cell related
operations you want. We recommend you to use this utility class because
the utility class will look for existing `CellStyle` object which equal
to the new style to reuse first. If no existing style matches, it just
create new one. It will also skip those cells that have equal style as
new style. So you don't have to check by yourself. This can avoid
creating redundant `CellStyle`.

**Change style example**

``` java
Range selection = Ranges.range(spreadsheet.getSelectedSheet(), spreadsheet.getSelection());

//change horizontal alignment
CellOperationUtil.applyAlignment(selection, Alignment.CENTER);
//change vertical alignment
CellOperationUtil.applyVerticalAlignment(selection, VerticalAlignment.TOP);

//change border
CellOperationUtil.applyBorder(selection, ApplyBorderType.EDGE_TOP
                                , BorderType.THIN, "#FF00FF");
```

All methods of `CellOperationUtil` require a Range object. You can use
Ranges to select one or more cells. In this example, we get the current
user-selected cells and pass it to `CellOperationUtil.applyAlignment()`.
Then `CellOperationUtil` will do those details stuffs for us to change
horizontal alignment.

## Using `Range` API

Althought the utility class
(<javadoc directory="keikai">io.keikai.api.CellOperationUtil</javadoc>)
provides convenience, but it doesn't provide complete API to change all
properties for a style. Sometimes you still need to use `Range` API.

Steps to change the style of a range:

1.  Clone its `CellStyle` object
2.  Set new value on the cloned `CellStyle`
3.  Set it back to the original `Range` object.

The following codes demonstrate how to change alignment:

``` java
    public void applyAlignment() {
        Range selection = Ranges.range(ss.getSelectedSheet(), ss.getSelection());
        CellStyle oldStyle = selection.getCellStyle();
        EditableCellStyle newStyle = selection.getCellStyleHelper().createCellStyle(oldStyle);
        newStyle.setAlignment( (Alignment)hAlignBox.getSelectedItem().getValue());
        selection.setCellStyle(newStyle);
    }
```

  - Line 4:
    <javadoc directory="keikai">io.keikai.api.Range.CellStyleHelper</javadoc>
    is a utility class that can you clone style related object and
    returns an editable version such as
    <javadoc  directory="keikai">io.keikai.api.model.EditableCellStyle</javadoc>
    or
    <javadoc directory="keikai">io.keikai.api.model.EditableFont</javadoc>.
  - Line 5: Change the style on newly-created cell style object.
  - Line 6: Set newly-created cell style object back to range to apply
    change.

# Example

The example application can display a cell's alignment and border status
and let you change the alignment of one or multiple cells. ![
center](/assets/images/dev-ref/zss-essentials-cellStyle-alignment.png " center")

``` java
public class CellStyleComposer extends SelectorComposer<Component> {

    @Wire
    private Label cellRef;
    @Wire
    private Label hAlign;
    @Wire
    private Label vAlign;
    @Wire
    private Label tBorder;
    @Wire
    private Label bBorder;
    @Wire
    private Label lBorder;
    @Wire
    private Label rBorder;
    @Wire
    private Listbox hAlignBox;
    @Wire
    private Listbox vAlignBox;
    @Wire
    private Spreadsheet ss;

    @Listen("onCellFocus = #ss")
    public void onCellFocus() {
        CellRef pos = ss.getCellFocus();
        refreshCellStyle(pos.getRow(), pos.getColumn());
    }

    private void refreshCellStyle(int row, int col) {
        Range range = Ranges.range(ss.getSelectedSheet(), row, col);

        cellRef.setValue(Ranges.getCellRefString(row, col));

        CellStyle style = range.getCellStyle();

        // display cell style
        hAlign.setValue(style.getAlignment().name());
        vAlign.setValue(style.getVerticalAlignment().name());
        tBorder.setValue(style.getBorderTop().name());
        bBorder.setValue(style.getBorderBottom().name());
        lBorder.setValue(style.getBorderLeft().name());
        rBorder.setValue(style.getBorderRight().name());

        // update to editor...

    }

    @Listen("onSelect = #hAlignBox")
    public void applyAlignmentByUtil() {

        Range selection = Ranges.range(ss.getSelectedSheet(), ss.getSelection());
        CellOperationUtil.applyAlignment(selection
                , (Alignment)hAlignBox.getSelectedItem().getValue());
    }

    @Listen("onSelect = #vAlignBox")
    public void applyVerticalAlignmentByUtil() {

        Range selection = Ranges.range(ss.getSelectedSheet(), ss.getSelection());
        CellOperationUtil.applyVerticalAlignment(selection
                , (VerticalAlignment)vAlignBox.getSelectedItem().getValue());
    }

    //omitted codes...
}
```

  - Line 38\~43: Get various style information from `CellStyle`
  - Line 53,61: Apply alignment with `CellOperationUtil`
