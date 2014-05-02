# Breaks from Zend Framework 2

This is a list of breaks from the standard ZF2 method of doing things. All documentation
in ZF2 should be valid unless stated here.

Module dependency checking - Forge is using custom module dependency checking. This is
    only because, at the time of this writing, dependency checking does not appear
    to actually be in ZF2 master. ZF2 dependency checking will very likely replace
    Forge's.

ViewTemplateMap - Since Forge is using designs and areas, this service is somewhat
    invalidate. It still functions exactly as specified, but it would never make
    sense to directly specify a template without design and area considered.

    This will still have a use in Forge Framework in the future. We will use it
    as a caching mechanism for resolved paths. Once a path is resolved by the
    ViewTemplatePathStack, we can store it in the template map, then eventually
    in a file at the end of the render event. This will, of course, have an on/off
    setting so that it doesn't impede development.

View rendering - Forge uses a composite view rather than Zend's standard view strategy.

    The renderer was updated to allow child view models to render, even without a
    template set, so long as they have children.

    An event listener was created to inject simple views into a composite view. It
    uses the identifier "simple_view_content", so do not use this identifier
    within your code. The simple view is injected as a child block for
    "main_content_children". This does not affect view model types other than
    ViewModel; JsonModel, FeedModel, and ConsoleModel work as normal.

ExceptionStrategy - Zend's ExceptionStrategy handles rendering of exceptions. In
    Forge, this responsibility was moved up to a default exception handler. The
    exception strategy now simply throws error type exceptions rather than rendering
    them.

Translator - The translator was updated to log untranslated strings. The translator
    was also updated to cascade translations on top of existing translations, rather
    than replace them. This allows for simpler translation file management.

Layout view helper - The layout view helper is no longer valid since the view is
    composite. It would not make sense for any single block in the view to be able
    to change the overall template.

Layout controller plugin - The layout controller plugin will have to be called
    after the getCompositeView(). This is because the getCompositeView() sets the
    layout based on the config if a template is set on the root node. This will
    override any previous settings. To make this work, no matter the order, we
    would have to put some type of flag in place to notify us that the template
    has been changed. Comparing it to the original is not enough, because users
    may be intending to dynamically set to template in the controller back to the
    default because they know that the view config has changed it
