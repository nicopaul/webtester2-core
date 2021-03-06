[Home](../README.md)

# @Produces
This annotation can be used on methods of `PageFragment` sub-classes in order to trigger the creation and firing of an `Event` whenever that method is executed.

**Constraints:**
- Only `Event` classes extending `AbstractPageFragmentEvent` are supported
- Only works in `PageFragment` classes, not `Page`
- Used `Event` classes need to define a `PageFragmentEventBuilder` inner class


## PageFragmentEventBuilder
Instances of this interface are used to build `Event` instances when `@Produces`
is used. The implementation classes need to be declared as static inner classes
of the `Event` named `Builder`. This convention allows for the dynamic used of
these builders without having to manage all builder types manually.

```java
public class ClickedEvent extends AbstractPageFragmentEvent {

    public ClickedEvent(PageFragment fragment) {
        super(fragment);
    }

    @Override
    public String describe() {
        return "clicked: " + getPageFragmentName();
    }

    public static class Builder extends AbstractPageFragmentEventBuilder<ClickedEvent> {

        @Override
        protected ClickedEvent buildWith(PageFragment fragment) {
            return new ClickedEvent(fragment);
        }

    }

}
```

In cases where data from the `WebElement` is needed to build the `Event` there
are two hooks which are called by WebTester before and after invoking the annotated
method. These are only used by the framework if the builder specifies that it
needs the data.

```java
public class TextSetEvent extends AbstractPageFragmentEvent {

    private final String before;
    private final String after;

    public TextSetEvent(PageFragment fragment, String before, String after) {
        super(fragment);
        this.before = before;
        this.after = after;
    }

    @Override
    public String describe() {
        return "text of '" + getPageFragmentName() + "' was set to '" + after + "' (was '" + before + "')";
    }

    public static class Builder extends AbstractPageFragmentEventBuilder<TextSetEvent> {

        private String before;
        private String after;

        @Override
        public boolean needsBeforeData() {
            return true;
        }

        @Override
        public PageFragmentEventBuilder<TextSetEvent> setBeforeData(WebElement webElement) {
            this.before = webElement.getAttribute("value");
            return this;
        }

        @Override
        public boolean needsAfterData() {
            return true;
        }

        @Override
        public PageFragmentEventBuilder<TextSetEvent> setAfterData(WebElement webElement) {
            this.after = webElement.getAttribute("value");
            return this;
        }

        @Override
        protected TextSetEvent buildWith(PageFragment fragment) {
            return new TextSetEvent(fragment, before, after);
        }

    }

}
```

# Linked Documentation

- [Page Fragments](page-fragment.md)
