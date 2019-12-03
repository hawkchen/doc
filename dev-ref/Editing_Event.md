# Editing Cell Events

There are three editing events that Keikai spreadsheet supports:

## onStartEditing

This event is fired only once at the moment when a user presses the
first key to start editing. When the corresponding event listener is
invoked, a
<javadoc directory="keikai">io.keikai.ui.event.StartEditingEvent</javadoc>
object is passed as an argument. This event allows you to cancel the
edit action or change edit value.

## onEditboxEditing

This event is fired when a user is editing a cell and it is similar to
Textbox component's onChanging event. When the corresponding event
listener is invoked, a
<javadoc directory="keikai">io.keikai.ui.event.EditboxEditingEvent</javadoc>
object is passed as an argument.

## onStopEditing

This event is fired when a user has finished editing a cell. It is
identified by the user hitting the enter key or clicking outside of the
editing cell. When the corresponding event listener is invoked, a
<javadoc directory="keikai">io.keikai.ui.event.StopEditingEvent</javadoc>
object is passed as an argument. This event allows you to cancel the
edit action or change edit value.

Pressing "delete" key can also change the cell content, but you have to
listen [ Key
Event](ZK_Spreadsheet_Essentials/Working_with_Spreadsheet/Handling_Events/Key_Event "wikilink")
for that action.

# Event Monitor Example

We still use previous "Event Monitor" application demonstrate event
listening.

![ center](/assets/images/dev-ref/zss-essentials-events-filter.png " center")

When we type the word "test" in A1 cell, the informations of
corresponding events sent to the server are displayed in the panel:

1.  Start editing A1...
      -   
        When we press "t", the onStartEditing event is sent. However,
        the typing value is still not saved in Spreadsheet's data model,
        so the editing value is empty. The client value is "t" which is
        the same as what we just type.
2.  Editing A1...
      -   
        There are 4 lines started with "Editing A1". Each time we press
        a key to edit, onEditboxEditing event is sent and the first
        onEditboxEditing is sent just right after onStartEditing.
3.  Stop editing A1...
      -   
        The onStopEditing event is sent when we press the enter key, and
        you can see editing value now is the same as A1's text.

Next, we present you how to listen these events and print messages with
related data in a controller.

``` java
public class EventsComposer extends SelectorComposer<Component>{
    //omitted codes...

    @Listen("onStartEditing = #ss")
    public void onStartEditing(StartEditingEvent event){
        StringBuilder info = new StringBuilder();
        String ref = Ranges.getCellRefString(event.getRow(),event.getColumn());
        info.append("Start editing ").append(ref)
        .append(", editing-value is ").append("\""+event.getEditingValue()+"\"")
        .append(" client-value is ").append("\""+event.getClientValue()+"\"");
        
        //...
    }

    @Listen("onEditboxEditing = #ss")
    public void onEditboxEditing(EditboxEditingEvent event){
        StringBuilder info = new StringBuilder();
        String ref = Ranges.getCellRefString(event.getRow(),event.getColumn());
        info.append("Editing ").append(ref)
        .append(", value is ").append("\""+event.getEditingValue()+"\"");
        
        //...
    }   
    
    @Listen("onStopEditing = #ss")
    public void onStopEditing(StopEditingEvent event){
        StringBuilder info = new StringBuilder();
        String ref = Ranges.getCellRefString(event.getRow(),event.getColumn());
        info.append("Stop editing ").append(ref)
        .append(", editing-value is ").append("\""+event.getEditingValue()+"\"");
        
        //...
    }
```

  - Line 4,15,25: Apply `@Listen` to listen an event with the syntax
    `[EVENT NAME] = [COMPONENT SELECTOR]`. All event name can be found
    in <javadoc directory="keikai">io.keikai.ui.event.Events</javadoc>.
    The "\#ss" is the component selector which means the component with
    id "ss" on the ZUL page. (SelectorComposer supports various selector
    syntax that let you select components easily. Please refer to [ZK
    Developer's Reference/MVC/Controller/Wire
    Components](ZK_Developer's_Reference/MVC/Controller/Wire_Components "wikilink"))
    .

<!-- end list -->

  - Line 7: The `getRow()` returns 0-based row index of the cell which
    is in editing and `getColumn()` returns column index. The cell A1's
    row and column index are both 0. We have not introduced
    <javadoc directory="keikai">io.keikai.api.Ranges</javadoc> formally
    yet, but you just treat it as a utility class that can help you
    convert row and column index into a cell reference, e.g. A1.
  - Line 9: The `getEditingValue()` returns the value stored in
    server-side's data model.
  - Line 10: The `getClientValue()` returns the value we type in the
    browser which might be different from editing value.

## Override Editing Value

In addition to displaying editing value, we can even override it. The
screenshot below demonstrate this case. There are several special cells
that contain "Edit Me". After we enter a word "test" in D3, it turns to
be "test-Woo". You can see the value changed from the right hand side
panel.

![ center ](/assets/images/dev-ref/zss-essentials-events-override-value.png " center ")

How do we make it? Just listen onStopEditing event and change the
editing value.

``` java

    @Listen("onStopEditing = #ss")
    public void onStopEditing(StopEditingEvent event){
        StringBuilder info = new StringBuilder();
        String ref = Ranges.getCellRefString(event.getRow(),event.getColumn());
        info.append("Stop editing ").append(ref)
        .append(", editing-value is ").append("\""+event.getEditingValue()+"\"");
        
        //...
        
        if(ref.equals("D3")){
            String newValue = event.getEditingValue()+"-Woo";
            //we change the editing value
            event.setEditingValue(newValue);
            addInfo("Editing value is changed to \""+newValue+"\"");
        }else if(ref.equals("E3")){
            //forbid editing
            event.cancel();
            addInfo("Editing E3 is canceled");
        }
    }
```

  - Line 13: Override editing value with new value.
  - Line 17: The `cancel()` can cancel this editing, and nothing will be
    saved to the cell.

# onAfterCellChange

This event is fired when you change the contents or styles of one or
more cells directly or indirectly. Therefore, it is triggered by user
editing or calling `Range` API. If you edit a cell, this event is fired
after `onStopEditing` event. When the corresponding event listener is
invoked, a
<javadoc directory="keikai">io.keikai.ui.event.CellAreaEvent</javadoc>
object is passed as an argument. This event only tells you which range
of cells change but doesn't tell you which part (value or style) the
cells change.

## Event Monitor Example

Let's get back to our event monitor example to see when
onAfterCellChange is sent. ![
center](/assets/images/dev-ref/zss-essentials-events-cellChange.png " center")

According to the screenshot above, when we enter "abc" in A11 or change
background color in A12:C13, cell changes event are sent. Let us see
source code about listening this event:

``` java
public class EventsComposer extends SelectorComposer<Component>{
    //omitted codes...
 
    @Listen("onAfterCellChange= #ss")
    public void onAfterCellChange(CellAreaEvent event){
        StringBuilder info = new StringBuilder();
        
        info.append("Cell changes on ")
        .append(Ranges.getAreaRefString(event.getSheet(), event.getArea()));
        info.append(", first value is \""
        +Ranges.range(event.getSheet(),event.getArea()).getCellFormatText()+"\"");
        
        //...
    }
```

  - Line 4, 5: Specify onAfterCellChange in `@Listen` and apply it to a
    method. A
    <javadoc directory="keikai">io.keikai.ui.event.CellAreaEvent</javadoc>
    object is passed in when the method is invoked.
  - Line 9: `getArea()` return an
    <javadoc directory='zss'>io.keikai.ui.Rect</javadoc> object to
    indicate the area where the change event happens, but we need to
    convert the object to readable area reference like B2:B2 with a
    utility method of Ranges.
