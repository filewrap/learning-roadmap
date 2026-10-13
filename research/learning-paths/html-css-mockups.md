# HTML, CSS, Mockups, and UI Design: A Practical Learning Chapter

## What this area is—and why it matters

A web page is both information and an interface. **HTML** (HyperText Markup Language) describes the document’s structure and meaning: headings, paragraphs, links, forms, images, and sections. A browser parses HTML into a document tree that software can inspect and render. **CSS** (Cascading Style Sheets) describes presentation: colors, type, spacing, borders, and layout. **UI design** (user-interface design) arranges visible controls and information so people can understand a page and accomplish tasks. A **mockup** is a visual proposal for a screen; a **wireframe** is an intentionally simplified blueprint focused on structure, content, navigation, and interactions. A **prototype** adds some simulated behavior so a flow can be reviewed. [1] [2][3]

Learning these together helps you move from an idea to a page that works across screen sizes and can be used by people with different needs. The repository plan for this topic recommends a progression from plain HTML/CSS and small static pages, through responsive layout and design-to-code practice, then design systems, animation, and an original mockup-to-implementation capstone. It names MDN, freeCodeCamp, The Odin Project, Flexbox Froggy, CSS Diner, Grid Garden, Figma’s beginner and wireframing materials, Frontend Mentor, and public systems such as Carbon, Ant Design, and Material as possible materials. Treat that list as a menu, not a required syllabus; a few well-chosen resources and repeated practice are enough to start. [4]

## Before you begin: prerequisites and vocabulary

You need only basic computer use: create and save text files, organize folders, and open a file in a browser. No JavaScript, framework, or design software is required to begin. A text editor and browser are sufficient for the first exercises. Use a design tool such as Figma or Penpot later if you want a shared visual canvas; the core concepts transfer between tools.

A **tag** is markup such as `<p>` or `</p>`. An **element** is the structure represented by a tag pair and its content. An **attribute** supplies information in a start tag, for example `href` in `<a href="about.html">About</a>`. A **selector** tells CSS which elements to style. A CSS **declaration** is a property and value, such as `color: navy`; a **rule** groups a selector with declarations. The **cascade** is the process by which the browser resolves competing CSS rules. The **box model** describes an element’s content, padding, border, and margin. A **breakpoint** is a condition at which a responsive layout changes. A **component** is a reusable interface piece such as a button or card. A **design token** is a named reusable value—such as a color or spacing size—used to keep a design consistent.

One more distinction is useful in practice: a browser’s rendered appearance is not the same thing as its document semantics. Assistive technology and browser features use the document structure and accessible names, not just what looks like a heading or button. Likewise, a mockup records an intention; it does not prove the implemented interaction, responsive behavior, or accessibility is correct. Treat the browser page as something to inspect and test, not just a picture to match. [1] [2]

## A sensible learning sequence

### 1. Structure first: HTML and meaning

Begin by making a complete, valid document. HTML elements form a nested tree; keep elements properly nested, and use the element whose meaning matches the content. For example, use headings for headings, paragraphs for prose, lists for groups of items, links for navigation, and buttons for actions. Semantic HTML is not merely a naming preference: native elements supply useful behavior and information to browsers and assistive technologies. A real `<button>` can be reached and activated by keyboard without recreating that behavior on a generic `<div>`. [1] [2]

A minimal page:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Garden Notes</title>
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <header>
      <h1>Garden Notes</h1>
      <nav aria-label="Main navigation">
        <a href="#plants">Plants</a>
      </nav>
    </header>
    <main id="plants">
      <h2>What is growing</h2>
      <p>Three herbs are ready to harvest.</p>
      <button type="button">Add a note</button>
    </main>
  </body>
</html>
```

The `viewport` metadata lets mobile browsers use the device viewport for page layout. `header`, `nav`, and `main` give meaningful structure; the heading levels describe an outline rather than a font-size choice. A link moves somewhere; a button performs an action. Add text alternatives to informative images, and give form controls visible labels. These small decisions support people navigating with keyboards or screen readers and make the page easier to maintain. [1] [2]

For forms, connect a visible `<label>` to its control with matching `for` and `id` values, or nest the control inside the label. A placeholder is a hint, not a durable substitute for a label: it can disappear during entry and may not explain the field’s purpose. Group related choices with `<fieldset>` and `<legend>`. If a submission fails, identify the field and explain how to correct it in text rather than relying on a red border alone. Keep forms as short as the task allows; unnecessary questions increase effort. [9]

### 2. CSS foundations: rules, cascade, and boxes

CSS is a separate language for styling HTML. A rule has a selector and declarations. For example:

```css
body {
  color: #20252b;
  font-family: system-ui, sans-serif;
  line-height: 1.5;
}

button {
  padding: 0.6em 1em;
  border: 0;
  border-radius: 0.4rem;
  background: #155eef;
  color: white;
}
```

When rules conflict, the browser considers such things as origin, importance, selector specificity, and source order. Avoid trying to fix every conflict with increasingly specific selectors or `!important`; understand which rule wins and prefer simple, predictable selectors. The box model is equally foundational: content sits inside padding, then border, then outside margin. `box-sizing: border-box` makes declared dimensions include padding and border, which often makes sizing easier to reason about. MDN’s CSS guides are useful references for the cascade, specificity, inheritance, units, and box model. [3]

When a style does not appear, inspect the element’s computed styles in developer tools: confirm the selector matches, check crossed-out declarations and inherited values, and inspect the winning rule and its source. This is faster than adding another override. A common starter reset is `*, *::before, *::after { box-sizing: border-box; }`; it makes component widths easier to calculate, but it does not remove the need to account for margins, intrinsic content, or min/max constraints. Start with a small type and spacing scale (for example, `0.5rem`, `1rem`, `1.5rem`, `2rem`) and named custom properties when repeated values emerge, instead of accumulating near-duplicates.

Start with readable type, comfortable line length, spacing, and clear visual hierarchy before decorative effects. The page should still make sense if colors or fonts change. Use CSS classes for reusable visual roles (for example `.card`), not a new one-off rule for every element.

### 3. Layout: choose the right tool

Normal document flow stacks block content and places inline content within text. Learn flow before reaching for positioning. **Flexbox** is a one-dimensional layout model: it distributes and aligns items along a row or a column. It is well suited to a navigation bar, a row of controls, or content with flexible alignment. **Grid** is two-dimensional: it defines rows and columns together, making it useful for page regions and card collections. They work well together; neither is a universal replacement for the other. [5] [6]

```css
.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  align-items: center;
}

.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 15rem), 1fr));
  gap: 1rem;
}
```

The first rule lays out action controls on one main axis and lets them wrap when space is tight. The second creates as many columns as fit while keeping each card from becoming too narrow; `minmax()` expresses a minimum and flexible maximum track size. Try Grid for repeated cards, Flexbox for a linear group, and inspect the result at different widths. Learn alignment, `gap`, `flex-wrap`, grid tracks, and flexible units before memorizing elaborate shorthand. [5] [6]

Choose based on the relationship you need to express: use Flexbox when items form a line or wrapped set and their sizes should respond to available space; use Grid when rows and columns need coordinated tracks. In the card example, `min(100%, 15rem)` prevents a card’s minimum from exceeding a very narrow container, while `auto-fit` packs the available tracks. If cards still overflow, look for long unbreakable strings, fixed-width children, or a grid item’s intrinsic minimum—not just the gap. Try `minmax(0, 1fr)` for a track that must shrink below its content’s min-content width, and add appropriate wrapping such as `overflow-wrap: anywhere` for exceptional long identifiers. Inspect the grid overlay in browser developer tools to see tracks and gaps. [6]

### 4. Responsive design and robustness

Responsive design means designing for the available space and user context rather than assuming one fixed screen. Start with content and a flexible layout, then introduce a media query when the layout actually needs to change. Avoid choosing breakpoints solely because a named phone or tablet has a particular width: resize the browser and find where the content becomes cramped. Use relative units and flexible tracks where they suit the job, allow text to wrap, and check both narrow and wide views. The repository plan points learners to web.dev’s responsive-design course and MDN’s layout material for continued study. [3] [4]

```css
.page {
  width: min(100% - 2rem, 68rem);
  margin-inline: auto;
}

@media (min-width: 48rem) {
  .content-layout {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 2rem;
  }
}
```

The container remains within the viewport while gaining a readable maximum width; at a wider available size, the content can split into two columns. This is a starting point, not a magic device-specific rule. Test long words, enlarged text, and real content: placeholder text often hides layout problems.

Use a breakpoint only when the content or task stops working well—for instance, a navigation row collides or a reading column becomes too long—not because a design is labeled “tablet.” Fluid layout methods often cover intermediate sizes without a query; use a query to make a deliberate change in layout or presentation. Also make media fluid (`img { max-width: 100%; height: auto; }`) to avoid overflow, and provide intrinsic `width` and `height` attributes on images so the browser can reserve their aspect ratio before loading. That helps avoid layout shifts. A single fluid image may still waste bandwidth or crop poorly on some screens; responsive image sources (`srcset`, `sizes`, or `<picture>`) are a later optimization when alternate resolutions or crops are available. [10] [11]

### 5. Accessibility is part of the design, not final polish

Accessibility means that people with different sensory, motor, and cognitive abilities can perceive and operate the interface. Semantic structure, keyboard operation, useful link text, labels, text alternatives, visible focus, and sufficient contrast belong in the first draft. WCAG 2.2 includes requirements such as not relying on color alone and minimum contrast for text: generally 4.5:1 for normal text and 3:1 for large text at Level AA, subject to stated exceptions. These are specific conformance criteria, not a complete substitute for testing with people or assistive technology. [1] [7]

Do not remove a focus outline just because it looks plain; provide a clear focus style if customizing it. Do not communicate “error” only with red; include wording or another indicator. Make sure content can be enlarged without clipping. CSS visual reordering should not contradict the meaningful reading and keyboard order of the HTML. Accessibility is easier when the underlying structure and behavior are sound. [1] [7]

Make the checks concrete: tab through the page and confirm both that focus is visible and that the sequence follows the task; activate controls with Enter or Space as appropriate; zoom text to 200%; and check a narrow viewport for two-direction scrolling or clipped content. WCAG 2.2 AA also includes a 24-by-24 CSS-pixel minimum pointer target size with listed exceptions, so give small adjacent controls enough size or separation rather than assuming a visually neat icon is easy to hit. Automated contrast and markup checkers can find some problems, but cannot judge whether instructions make sense or whether a real task is understandable. [7]

### 6. Mockups and design-to-code

A useful workflow is: define the user’s task; sketch or wireframe the screen; make a visual mockup; test the flow; implement; compare the browser result with the intended design; revise. A wireframe keeps early discussion focused on layout, content, navigation, and key interactions. A high-fidelity mockup adds more visual detail, but it is still a proposal rather than a functioning implementation. Figma’s guide describes low-, mid-, and high-fidelity wireframes and emphasizes matching the level of detail to the question being explored. [8]

When translating a mockup, do not copy pixels blindly. Identify repeated patterns, content priority, spacing relationships, and which parts must adapt. Make a small style sheet of color, type, spacing, and corner-radius choices. A design system organizes shared rules and components; even a tiny project benefits from consistent button, card, and heading styles. The scope plan names Carbon, Ant Design, and Material as systems worth studying once you can read their component and token conventions. [4]

Before polishing, state the user, their goal, and the main action; then sketch the information order and a simple task path. Review a low-fidelity wireframe for missing content and confusing navigation, not color nuance. In a higher-fidelity pass, define a few shared values (for example, `--space-2`, `--color-text`, `--radius-card`) and apply them consistently. Compare the implementation with the mockup at more than one viewport and with actual text: a screenshot match at one size is not a successful translation if content cannot grow, focus cannot be seen, or the primary action is ambiguous. A design system is a set of reusable guidance and components, not a reason to adopt every component; select patterns that solve the project’s needs and verify their states and accessibility in context. [8]

### 7. Polish and advanced topics

Once basic pages work, study pseudo-classes such as `:hover` and `:focus-visible`, transitions, transforms, and keyframe animation. Add motion only when it helps communicate a state or relationship; ensure essential information does not depend on animation. Then consider a framework or component library if a project benefits from shared prebuilt patterns. Frameworks do not replace HTML semantics, CSS layout understanding, responsive testing, or accessibility. The repository plan suggests practicing with plain HTML/CSS first, then exploring Bootstrap or Tailwind, and later reading a design system’s documentation and source. Tool APIs change faster than semantic HTML, the cascade, and layout fundamentals, so verify current documentation when adopting a tool. [3] [4]

## Practice tasks and answer checks

1. **Build a semantic profile page.** Include a page title, one `h1`, two `h2` sections, a list of interests, a navigation link, and a button. **Check:** headings form a sensible outline; the link navigates; the button is a button element; no content is represented only by repeated line breaks.

2. **Style a readable card.** Add a class to one section and set text color, background, padding, border, and a comfortable line height. **Check:** the card has visible separation from its surroundings, text is readable, and changing the card’s class rule updates the card without affecting unrelated content.

3. **Arrange three cards.** Use Grid with a gap, then resize the browser. **Check:** columns share available width, cards do not overlap, and at a narrow viewport they fit without horizontal scrolling. If they do not, revise the minimum track size or layout.

4. **Add a navigation row.** Use Flexbox and `gap`; allow wrapping. **Check:** items align cleanly, wrap rather than collide at narrow widths, and keyboard focus is visible as you press Tab.

5. **Sketch a small event page before coding.** Draw a low-fidelity wireframe with title, date, venue, main action, and supporting information; then build it and compare. **Check:** a new viewer can identify the event and its main action quickly, the information order is consistent, and the narrow layout remains usable. Ask someone to find the date and action without coaching; note where they hesitate.

6. **Accessibility pass.** Navigate the page with keyboard only, inspect image alternatives and form labels, and check text/background contrast. **Check:** all actions can be reached and activated, focus is visible, color is not the only signal, and ordinary text meets the WCAG AA contrast threshold where applicable. [1] [7]

7. **Form and failure-state pass.** Add a short contact form with two labeled fields and a submit button; sketch what the user sees when a required value is missing. **Check:** every field has a persistent visible label, related choices have a group label, and an error is identified in text with a correction hint. Try keyboard-only completion; do not assume a placeholder or color alone communicates the instructions. [9]

8. **Responsive content stress test.** Put a long title, a long unbroken URL, and an image in the page; test at a narrow width, at 200% text zoom, and with a wider desktop viewport. **Check:** no essential content is clipped or requires horizontal scrolling, image proportions remain correct, cards wrap or tracks shrink intentionally, and a breakpoint appears only where the content benefits. Use developer tools to find the element that causes overflow rather than hiding it with `overflow-x: hidden`. [10] [11]

## Common mistakes to avoid

- Using `<div>` for everything, or choosing elements only for their default appearance. Choose meaning first; style second. [1]
- Treating HTML as a visual drawing language. HTML describes content and structure; CSS controls presentation. [2] [3]
- Using fixed widths and heights everywhere. Real text, translations, zoom, and small screens can overflow them.
- Reaching for absolute positioning before understanding normal flow, Flexbox, or Grid. Positioning can overlay content and is usually not the right tool for a whole-page layout.
- Assuming Flexbox and Grid are competitors. Choose based on one-dimensional versus two-dimensional layout needs. [5] [6]
- Accumulating overrides and `!important`. Reduce conflicting rules, use clear class names, and inspect computed styles in browser developer tools.
- Styling only the “happy path.” Check longer content, narrow widths, keyboard focus, contrast, and text zoom.
- Treating a placeholder as a form label or treating a mockup’s static appearance as evidence that its error, focus, and success states work. [9]
- Fixing overflow by hiding it. Find the fixed width, long word, intrinsic grid minimum, or media element causing it, then choose a content-appropriate fix. [6] [10]
- Making a mockup so polished that feedback shifts to decoration before the structure and task flow have been tested. Lower fidelity when you need structural feedback. [8]
- Copying a design screenshot without considering semantics, responsive behavior, or whether the visible controls actually work.

## A concise next-step sequence

1. Work through MDN’s HTML and CSS learning guides and build one small semantic page. [1] [3]
2. Practice selectors, the box model, normal flow, Flexbox, and Grid; use the MDN concept guides when stuck. [3] [5][6]
3. Make the page responsive and keyboard-usable; test real content, image sizing, 200% text zoom, target sizes, and contrast. Use WCAG as a conformance reference, not a promise of complete accessibility. [7] [10]
4. Follow one structured free practice path named in the repository plan—such as The Odin Project Foundations or freeCodeCamp Responsive Web Design—and build several small pages without frameworks. [4]
5. Create a wireframe in a design tool, then implement the same page and compare it. Figma’s beginner tutorials and wireframing guide are optional tool-specific support; the workflow is transferable. [4] [8]
6. Take on a design-to-code challenge, study one public design system, and finish with an original multi-page mockup-to-code project. Add a framework only when you can explain what it saves and what it abstracts. [4]

## References

[1]: https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/HTML "HTML: A good basis for accessibility — MDN"
[2]: https://html.spec.whatwg.org/ "HTML Living Standard — WHATWG"
[3]: https://developer.mozilla.org/en-US/docs/Web/CSS "CSS: Cascading Style Sheets — MDN"
[4]: https://www.freecodecamp.org/learn/responsive-web-design/ "Responsive Web Design — freeCodeCamp (named in the supplied repository plan)"
[5]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox "Basic concepts of flexbox — MDN"
[6]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout "Basic concepts of grid layout — MDN"
[7]: https://www.w3.org/TR/WCAG22/ "Web Content Accessibility Guidelines (WCAG) 2.2 — W3C"
[8]: https://www.figma.com/resource-library/what-is-wireframing/ "What is wireframing? — Figma Resource Library"
[9]: https://www.w3.org/WAI/tutorials/forms/ "Forms Tutorial — W3C Web Accessibility Initiative"
[10]: https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Responsive_Design "Responsive design — MDN Learn web development"
[11]: https://web.dev/articles/responsive-web-design-basics "Responsive web design basics — web.dev"
