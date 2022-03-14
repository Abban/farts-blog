---
title: "Accessibility Workshop Notes"
date: 2022-03-14T00:00:01+01:00
draft: false
description: "Personal notes from an accessibility workshop I attended."
post_type: "blog"
tags: [ "Professional" ]
---

My thoughts in *italic*.

## Intro

### Basics

Affects way more people that you might think.

* *A11y is not an accessible name.*
* 15% of the world have disabilities *Fuck, we work harder on IE support than we do on accessibility*
* Auditory, Cognitive, Neurological, physical, speech, visual *Can be permanent or temporary, must be much harder for them*
* Old people.
* *Check out [WebAIM 1 Million Project](https://webaim.org/projects/million/).*
* 97% of web pages have issues. Contrast *JFC is this still a thing!?*, alt text, input labels, doc language, shitty buttons.

### Assistive Tech *(Screen readers etc)*

*Seems to be a bit software engineers think tech will solve everything.*

* See Sound by Wavio. *Seems like a privacy nightmare, plus AI driven so I'd bet there's also built in diversity issues.*
* Tobbi by Dynavox. *This looks more like something that had some actual research into it*

Bunch of other less Silicon Valley tools and apps follow. Look at the slides.

* SuperNova, Dolphin. *Some people need massive magnification*
* Screen Readers. *Document Semantics are super important, correct heading nesting etc. We don't test with in the same way people who actually use them use them. Again, think about document semantics all the time, **SUPER IMPORTANT**.*
* *Check out the browser inspector accessibility tree.*
* [a11ysupport.io](https://a11ysupport.io) has accessibility support matrixes. Screen readers are as weird with support as browsers.
* Navigating by heading is by far the most used way people use screen readers to find information.
* *Screen readers seem to suck.*

## Accessibility Testing

### Automated testing for accessibility.

* *Can only see 30% of issues so... worthwhile?*
* *Really sounds like they're not that valuable as tests, might even have a negative impact. Very brittle?*
* Look into VueAxe. Also axe-core. Also jest-axe.
* There is also an eslint plugin.

### Semi-Automated

* Axe Devtools *Not fully free*.
* Wave from WebAIM. Extension free not the automated tool.
* Tota11y.
* A11y.css.

Custom scripts

* *Examples were for elements like individual buttons, seems to be too micro managery.*

### Manual

Look in the slides.

* This might be the way to do things. An accessibility step in code reviews.
* Important to have no horizontal scroll on large zooms.
* Reduced motion options.
* Adaptive font sizes. Change font sizes and see if it updates, works.
* Semantic html again.
* Test with a screen reader.
* Image alts.
* High contrast mode.
* User tests.
* Also run tests in browser when dynamic elements are visible.

## Accessibility Patterns

How to write semantic html. You know this, use elements for their intended purposes.

* Output tag is important and not well known.
* ARIA is used to change the accessibility tree, but not the DOM.
* ARIA mainly adds roles and labels.
* ARIA Rules:
	1. When possible don't use it.
	2. Don't change native semantics wrap elements in divs with roles.
	3. Interactive elements must interact with a keyboard.
	4. Don't hide focusable elements.
	5. Name all interactive elements.
* Landmarks html5 are elements like main/header/nav/footer/section/form/search.
* Sort've html, defined using roles and the newer html5 elements.
* More than one role, then landmarks must have a label. `<aria-labelledby="id-of-element"/>`
* `aria-label` is for adding labels for screen readers without displaying them.
* Skip Links are used to provide a link to go straight to the main content. Anchor link points to the element.
* Decorative images should be hidden from the screen reader. Empty alt tag or `aria-hidden`.
* Icon buttons:
	* If has text prevent the icon being focusable.
	* If not use a hidden text element or `aria-label`. Native HTML > `aria-labelledby` > Hidden Element > `aria-label`.
* Read more links, basically UI elements with the same content:
	* Users with screen readers can't tell the context.
	* If you really need to use them use `aria-labelledby`.
* Breadcrumbs:
	* Nav and label them. Or add a hidden title element.
	* Current element has `aria-current`.
	* Add `aria-hidden` to separators.
* Toggle buttons:
	* Toggle `aria-pressed`.
	* Change label.
* Tooltips:
	* Make them buttons.
	* Use `aria-labelledby` pointing to the element with the tooltip text.
	* If the tooltip is descriptive use `aria-describedby`.
	* Add a hidden element to the button, similar to decorative images.
* Accordion:
	* Headers and sections. Button inside header.
		* `aria-controls` on button.
		* `aria-labelledby` on section.
		* Don't make icon focusable.
		* Add `aria-expanded` to buttons true/false.
		* Add `hidden` to closed sections.
	* Alternatively use `dt` and `dd` elements.
* Cards: *Not into this implementation*
	* Put them in a list.
	* Don't make the entire thing a link as it's a disaster for screen readers.
	* Make title the link.
	* Use css pseudo element and cover the element in the link clickable area. *Makes text not selectable!!* 😬
	* Progressive enhancement on hover/focus states.
* Notices: *Read more about this*
	* Mark them as dynamic with `role=status` or `aria-live`.
	* Use output tag.

### Form Validation

* Every form should be a form so they can be submitted by keyboard.
* Errors:
	* Remove browser defaults. They suck. `novalidate`.
	* Add `aria-invalid` to incorrect fields.
	* On the error text use `aria-describedby` pointing to the field.
	* Add `role=alert` on error message.
* Don't disable buttons when fields are invalid. Use `aria-disabled`.
* Focus first error on submit.

### SPA Routers

* Make heading 1 focusable.
* Focus on them after route change.

### Modals

Very complicated to make accessible. Research this more.

## WCAG Guidelines

POUR Principles:

* Perceivable: Should be perceivable by everyone.
	* Screen readable errors.
* Operable: Should be useable by everyone.
* Understandable
* Robust

Checklist [Here](https://a11yproject.com/checklist).

## TODO
* Learn how MacOS screen reader works and use it to navigate pages.
* Read the ARIA specification.

## Questions
* Is it helpful to add headings to site navigation? For people who navigate by heading. *YES*
* Crossover with SEO, because scrapers are similar to screen readers?
* What are the quickest wins we can achieve?
* How to navigate crazy forms like we have
