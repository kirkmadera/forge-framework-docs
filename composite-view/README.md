# Composite View

This document describes Forge's composite view

Forge introduces a composite view. This is a view built in parts by many individual
"mini controllers" called blocks. This helps to make blocks reusable, makes controllers
very thin, and makes local modifications to core and community code possible. It
also helps keep a local project with modifications more upgrade friendly.

The composite view plays well with modules not coded for it. The original template
paths remain in place, so template rendering is not broken if the module does
not know about areas and designs. Simple views returned by controllers as array,
null, or ViewModel are processed as usual by the standard ZF2 listeners that
create a view model, if needed, then attach a template to it. Then, this view is
injected into the composite view model as a child of "main_content_children", using
the identifier "simple_view_content". This is then rendered within the composite
view.
