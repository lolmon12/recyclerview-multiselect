#RecylerView Multiselect
RecyclerView MultiSelect is a tool to help implement single or multichoice selection on RecyclerView items. It does not provide the same interface as ListView's setChoiceMode(), but can do everything setChoiceMode() does. It is also small with a limited set of responsibilities, which means it should be suitable for coercing into off label uses.


<a href="http://www.methodscount.com/?lib=com.bignerdranch.android%3Arecyclerview-multiselect%3A%2B"><img src="https://img.shields.io/badge/Methods and size-core: 118 | deps: 14762 | 23 KB-e91e63.svg"></img></a>


### Basic Usage

The MultiSelector is the core object that manages multi-selection. However, you have to tell MultiSelector when to enter/leave selection mode.

```
MultiSelector multiSelector = new MultiSelector();
multiSelector.setSelectable(true); // enter selection mode
```

```
MultiSelector multiSelector = new MultiSelector();
multiSelector.setSelectable(false); // leave selection mode
```

You must also tell MultiSelector when an item has been selected.

```
MultiSelector multiSelector = new MultiSelector();
multiSelector.setSelectable(true); // enter selection mode
multiSelector.setSelected(myViewHolder, true); // set myViewHolder to selected
```

Your ViewHolder must implement the SelectableHolder interface. This library provides a default implementation called SwappingHolder that handles the visuals of being selected/unselected.


```
private MultiSelector mMultiSelector = new MultiSelector();

private class MyViewHolder extends SwappingHolder 
            implements View.OnLongClickListener { // (1)

    public MyViewHolder(View itemView) {
        super(itemView, mMultiSelector); // (2)
        itemView.setLongClickable(true);
    }

    @Override
    public boolean onLongClick(View view) { // (6)
        if (!mMultiSelector.isSelectable()) { // (3)
            mMultiSelector.setSelectable(true); // (4)
            mMultiSelector.setSelected(MyViewHolder.this, true); // (5)
            return true;
        }
        return false;
    }
}
```

Create a ViewHolder that subclasses SwappingHolder (1). MultiSelector works through the SelectableHolder interface to communicate to the ViewHolder when it has been selected/deselected. Therefore you must pass MultiSelector to SwappingHolder in the ViewHolder's constructor (2). Ensure MultiSelector is not already in selection mode (3), then enter selection mode (4). Finally set the item as selected (5). In this example, we chose to enter selection mode on long press (6). It is up to you to decide what triggers selection mode.

It is also up to you to notify MultiSelector when an item is clicked during selection mode. However, you will want to notify MultiSelector only during selection mode. When not in selection mode you can handle item clicks normally. 

```
private class MyViewHolder extends SwappingHolder implements View.OnClickListener {

    public MyViewHolder(View itemView) {
        super(itemView, mMultiSelector);
    }

    @Override
    public void onClick(View view) {
        if (mMultiSelector.isSelectable()) { // (1)
            boolean isSelected = 
                    mMultiSelector.isSelected(MyViewHolder.this.getAdapterPosition(), 
                            MyViewHolder.this.getItemId()); // (2)
            
            mMultiSelector.setSelected(MyViewHolder.this, !isSelected); // (3)
            
        } else { // (4)
            // do whatever we want to do when not in selection mode
            // perhaps navigate to a detail screen
        }
    }
}
```
If in selection mode (1), get the current state (2), and toggle it (3). If not in selection mode (4), perform the normal action.


Instead of performing steps 1-3 above, this library provides a convenience method called tapSelection(). If in selection mode, this method will toggle the selected state of an item and return true. If not in selection mode this method returns false. So the above code can be reduce to: 

```
private class MyViewHolder extends SwappingHolder implements View.OnClickListener {

    public MyViewHolder(View itemView) {
        super(itemView, mMultiSelector);
    }

    @Override
    public void onClick(View view) {
        if (!mMultiSelector.tapSelection(MyViewHolder.this)){
            // do whatever we want to do when not in selection mode
            // perhaps navigate to a detail screen
        }
    }
    
    ...
}
```

If you want to enter ActionMode as part of selection mode, it is your responsibility to do so. However this library provides a helper ActionMode.Callback helper class called ModalMultiSelectorCallback.


```
private MultiSelector mMultiSelector = new MultiSelector();

private ModalMultiSelectorCallback mActionModeCallback 
        = new ModalMultiSelectorCallback(mMultiSelector) { // (1)
    ...
};

private class MyViewHolder extends SwappingHolder
        implements View.OnClickListener, View.OnLongClickListener {

	...
	
    @Override
    public boolean onLongClick(View view) {
        if (!mMultiSelector.isSelectable()) {
            ((AppCompatActivity) getActivity()).startSupportActionMode(mActionModeCallback); // (2)
            mMultiSelector.setSelectable(true);
            mMultiSelector.setSelected(MyViewHolder.this, true);
            return true;
        }
        return false;
    }
}
```

First create an instance of ModalMultiSelectorCallback, passing in your MultiSelector (1). Then pass this callback into the system when starting ActionMode (2).

The ModalMultiSelectorCallback exposes all methods in the ActionMode.Callback interface that you would normally use to create/prepare/respond to/destroy the ActionMode menu items:

```
public boolean onCreateActionMode(ActionMode mode, Menu menu);

public boolean onPrepareActionMode(ActionMode mode, Menu menu);

public boolean onActionItemClicked(ActionMode mode, MenuItem item);

public void onDestroyActionMode(ActionMode mode);
```

For this example, we use want to delete items in selection/action mode:

```
private ModalMultiSelectorCallback mActionModeCallback
        = new ModalMultiSelectorCallback(mMultiSelector) {

    @Override
    public boolean onCreateActionMode(ActionMode actionMode, Menu menu) {
        super.onCreateActionMode(actionMode, menu);
        getActivity().getMenuInflater().inflate(R.menu.list_context_menu, menu);
        return true;
    }

    @Override
    public boolean onActionItemClicked(ActionMode actionMode, MenuItem menuItem) {
        if (menuItem.getItemId() == R.id.menu_item_delete) {
            actionMode.finish();

            for (int i = mObjects.size(); i >= 0; i--) {
                if (mMultiSelector.isSelected(i, 0)) { // (1)
                    // remove item from list
                    mRecyclerView.getAdapter().notifyItemRemoved(i);
                }
            }

            mMultiSelector.clearSelections(); // (2)
            return true;

        }
        return false;
    }
};
```

We can access each selected item (1) to remove it from our list. We also need to clear selection when finished removing all items (2).


##Download

[v2.0.4 AAR](http://repo1.maven.org/maven2/com/bignerdranch/android/expandablerecyclerview/2.0.4/expandablerecyclerview-2.0.4.aar)

**Gradle**

```
compile 'com.bignerdranch.android:expandablerecyclerview:2.0.4'
```

**Maven**

```
<dependency>
  <groupId>com.bignerdranch.android</groupId>
  <artifactId>expandablerecyclerview</artifactId>
  <version>2.0.4</version>
</dependency>
```


## Contributing


##License
