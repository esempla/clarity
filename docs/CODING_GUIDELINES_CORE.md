# Core Web Component Coding guidelines

#### Warning! We are still in the early stages of building out our initial set of Web Components and are not quite ready for contributions. Check back soon for updates.

To ensure the highest quality and maintainability for Clarity Web Components, we
adhere to the following guidelines for our Core Web Component codebase.

## Philosophy

* Web standards over frameworks.
* Lightweight, only ship what you need.
* Accessible by default.
* Minimally Stateful Components
* Automated/Build enforced code conventions and performance.

## Project Design

* Components should be as stateless as possible – leaning on the host application
  framework to manage state.

* Stateless utilities should default to being pure JavaScript functions instead
  of Classes.

* Stateful logic between components when necessary should be encapsulated in a
  Class following our Service conventions described in this document.

* Only Components can have side effect modules to allow auto-registration of the
  element.

* Core cannot take on any third-party dependencies other than the
  dependencies defined in the project already. Core also cannot have any
  dependencies on Clarity Angular. This restriction is to ensure proper tree
  shaking and performance standards.

* Core has the following dependencies:

  * [lit-element](https://lit-element.polymer-project.org/). Lit-element is a
    lightweight base class that provides syntactic sugar to Web Components.

  * [Ramda JS](https://ramdajs.com/) for common functional utilities. To use
    Currying, use the `curryN` function for TypeScript support.

## New Components

* New components should go through our standard [Development Contribution Guidelines](./../DEVELOPMENT_CONTRIBUTION.md).
  New components that have an accepted proposal and API design will need to
  follow the code conventions, as described in this outline. New components will
  need to have a new Storybook Story for the full API as well as any additional
  common use cases as additional stories.

## Code Conventions

* In general, if possible, any code convention should be enforced by the build
  system and not require explicit documentation. The following are conventions
  not currently enforced by our build.

## Filenames

* Components should end with a `element` suffix.
  `clr-core/my-toggle/my-toggle.element.ts`.

* Stateless Utilities should be named appropriately in the `utils` directory.
  `clr-core/common/utils/register.ts`.

* Stateful Utilities should be named with a `service` suffix in the `services`
  directory. `clr-core/common/services/common-strings.service.ts`.

* Common Interfaces should be named with a `interface` suffix in the `interfaces`
  directory. `clr-core/common/interfaces/types.interface.ts`.

* Common Enums should be named with a `enum` suffix in the `enums`
  directory. `clr-core/common/enums/key-events.enum.ts`.

* Common Functions should have an appropriate file name with no prefix or suffix
  that describes the functionality.

## Naming and TypeScript Conventions

* Function Utilities should be named by their single responsibility.
  `function myUtil() { }`

* Stateful Class Services should be suffixed with `Service`.
  `class MyUtilService { }`

* Private properties should be **not** prefixed with a `_` but denoted with the `private` keyword.
  Long term when EcmaScript [private fields](https://github.com/tc39/proposal-class-fields)
  and [private methods](https://github.com/tc39/proposal-private-methods) land
  we will migrate to use the native syntax.

  There are 2 exceptions to this rule, and they're the only cases where we allow
  the use of an `_` prefix: A private property is hidden behind a getter
  (and maybe a setter) that has the same name:

  ```typescript
  private _open: boolean;
  get open(): boolean;
  set open(value: boolean);
  ```

* Public properties and methods should not need `public` keyword as that is
  the default behavior.

  ```typescript
  open = false;
  next() { ... }
  private name: string;
  ```

* Constants internal to Clarity are named following the `SCREAMING_SNAKE_CASE`
  convention.

* Enum values are `SCREAMING_SNAKE_CASE` because they are constant. For
  instance: `KeyboardKeys.ARROW_DOWN`.

## Web Component Conventions

* Core Web Components are built using [lit-element](https://lit-element.polymer-project.org/)

* One Component per file.

* Custom Elements (Web Components) should be prefixed with `cwc-`.

* There is no prefix on public custom element attributes or properties.

* Prefer properties over attributes.

* Built-in component style options should be exposed via attributes and CSS
  Custom Properties.

* The Component class should be prefixed with `Cwc`. Example `export class CwcModal { }`

* Style attributes related to sizes should use t-shirt style values. Example
  `size="sm"` ... xs, sm, md, lg, xl xxl

* Events/props should follow Angular naming conventions. Example property `open`
  and event `openChange`.

* Components should rarely if ever expose public methods as this encourages state.

* Properties should only reflect into attributes if expecting primitive values
  like `string`, `number`, or `boolean`. Use the provided `@property` decorator
  from `@clr/core/common` as this provides the same lit-element decorator with
  our project defaults.

  ```typescript
  import { property } from '@clr/core/common';

  @property({ type: Boolean }) open = false;
  ```

* Custom events by default should not bubble events to the global document.

* Custom events should not re-emit events when an input property value changes.

* Custom events should be used to communicate to the application when a user
  interacts with the component UI.

* Custom events should use our built in `@event` decorator.

  ```typescript
  import { event } from '@clr/core/common';

  @event() private closedChange: EventEmitter<boolean>;

  this.closedChange.emit(true);
  ```

* Input properties should use our built in `@property` decorator which extends
  the lit-element decorator by adding our preferred defaults.

  ```typescript
  import { property } from '@clr/core/common';

  @property({ type: Boolean }) loadingState;
  ```

* If the component needs to render external text or HTML, use the [Slot API](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots).

  ```html
  <cwc-modal>
    <p>slot content</p>
  </cwc-modal>
  ```

* We do not use named slots as a public API but rather provide wrapper components.

  ```html
  <cwc-modal>
    <cwc-modal-header></cwc-modal-header>
    <cwc-modal-body></cwc-modal-body>
    <cwc-modal-actions></cwc-modal-actions>
  </cwc-modal>
  ```

  Internally we use named slots but automatically add the named slot.

  ```html
  <!-- cwc-modal template -->
  <slot name="header"></slot>
  <slot></slot>
  <slot name="actions"></slot>
  ```

  ```typescript
  // cwc-modal-actions
  connectedCallback() {
    super.connectedCallback();
    this.setAttribute('slot', 'actions');
  }
  ```

* Components should follow the appropriate TSDoc summary convention as well as
  appropriate TSDoc comments for properties and methods. See existing components
  for convention examples.

  ```typescript
  /**
   * Dropdown, example test component. Do not use in production.
   *
   * @element `cwc-test-dropdown`
   * @slot `default` - Content slot for dropdown content
   * @attr `outline` - Apply outline style.
   * @cssprop `--border-color`
   * @cssprop `--background`
   * @cssprop `--color`
   */
  export class CwcDropdown {}
  ```

* These component conventions allow frameworks to consume the Web easily
  Components. Here are a few examples.

  ```html
  <!--
    Example of a modal web component in Angular
    - `size` is a attribute style hook
    - [open] is setting a property on the element
    - (openChange) is listening for the `openChange` custom event
  -->
  <cwc-modal size="lg" [open]="true" (openChange)="log($event.detail)">
    <p>slot content</p>
  </cwc-modal>
  ```

  ```html
  <!--
    Example of a modal web component in Vue
    - `size` is a attribute style hook
    - :open is setting a property on the element
    - @openChange is listening for the `openChange` custom event
  -->
  <cwc-modal size="lg" :open="true" @openChange="log($event.detail)">
    <p>slot content</p>
  </cwc-modal>
  ```

  ```jsx
  {
    /*
    Example of a modal web component in Preact (React with React Shim coming soon)
    - `size` is a attribute style hook
    - open is setting a property on the element
    - openChange is listening for the `openChange` custom event
  */
  }
  <CwcModal size="lg" open={this.state.open} openChange={this.log}>
    <p>slot content</p>
  </CwcModal>;
  ```

## Component Templates

* Any button in your template should have `type="button"`, in case the component
  is used inside of a form.

* Never hard code strings in the template but use the `CommonStrings` service
  for i18n support.

## Component Styles

* Components should not have any external margins. Margins on a host element make
  assumptions of the layout that is external to their responsibility. Using a design
  token system, designers and developers should be able to layout components
  consistently and with explicit intent and constraint.

* Any style that causes layout reflow should **not** be exposed via CSS Custom Properties.

* Core components cannot rely on global styles such as clr-ui grid and typography.
  Components should be completely independent. This allows teams who are migrating
  from large existing code bases to adopt Clarity Core components incrementally without
  disruption to their existing codebase.

* Public API surface for component styling should **not** use the `clr-` global
  prefixing convention. If there is a naming collision with a local host var and
  global theming var it can break on the component ([see example](https://codepen.io/coryrylan/pen/vYEMxgo)).
  This is also necessary for micro frontends where a component may need to be
  aliased due to app embedding and e2e workflows. This prevents version
  collisions and dependencies form having to share the same global values.

* Components are themed using CSS Custom Properties defined on the `:host` element.
  This ensures the component is only styled explicitly by the API we define and
  cannot accidentally be changed at a global scope.

  ```scss
  // cwc-badge
  :host {
    // public style API
    --background: #{$clr-color-neutral-600};
    --color: #{$clr-color-on-neutral-600};

    // private styles below
  }
  ```

* Alternate states of a component should leverage existing base properties to
  keep the API small and flexible.

  ```scss
  // cwc-badge
  :host {
    --background: #{$clr-color-neutral-600};
    --color: #{$clr-color-on-neutral-600};
  }

  :host([status='success']) {
    --background: #{$clr-color-success-700};
    --color: #{$clr-color-on-success-700};
  }

  :host([status='danger']) {
    --background: #{$clr-color-danger-800};
    --color: #{$clr-color-on-danger-800};
  }
  ```

  This pattern allows devs to easily build their own states.

  ```scss
  // application level styles
  cwc-badge.product-badge {
    --background: purple;
    --color: white;
  }
  ```

  ```html
  <cwc-badge class="product-badge">1</cwc-badge>
  ```

* Host defined properties allow teams to prefix their own core components to
  share and embed with other products without the risk of a host app breaking
  their product specific styles.

  ```html
  <style>
    cwc-badge {
      --color: green;
    }
  </style>
  <cwc-badge></cwc-badge> <!-- only host application badges are changed -->

  <product-cwc-badge></product-cwc-badge> <!-- embedded product styles are preserved -->
  ```

## Public Import API

* Public Components are exported with a side effect file import.
* Public Utilities are only exported through the top-level import path.
  ```typescript
  import { CommonStrings } from '@clr/core'; // utilities
  import '@clr/core/button'; // component
  ```

## Usage Best Practices

* We ship modern es2015 modules as the default.
* Use a module bundler to dedupe and handle peer dependencies.
* Share peer deps to prevent multiple versions running at one time.
* `@clr/core/common` is private internal API only for internal use of Core components.

## Unit Testing

* We expect extensive unit test coverage of any code submitted. See existing
  components for test examples. The build enforces code coverage. You can
  run code coverage checks using the `core:` npm scripts defined in the
  `package.json`.

* Unit tests should not duplicate coverage. In particular, avoid multiple unit
  tests failing for the same error.

* Do not test several components at the same time, unless you're explicitly
  writing an integration test.

## Screenshot Testing

* Coming Soon...

## React/Preact Support

* Preact supports Web Components.

* Due to React not being compatible with Web Standards such as [custom elements](https://custom-elements-everywhere.com/)
  a shim layer needs to be created for every new component. More details on
  this will be added soon.

## Library Authors

* If you ship a library that contains Clarity Web Components, there are a few
  things to consider. First, custom element tag names are global to the DOM.
  To prevent collisions of the possibility of multiple versions running at the
  same time, you will need to alias the tag name. The alias name/prefix should
  be specific to your library.

  ```html
  <!-- default tag name -->
  <cwc-modal></cwc-modal>

  <!-- safe alias name to ship -->
  <my-lib-cwc-modal></my-lib-cwc-modal>
  ```

* When shipping a library using Clarity Web Components you must list
  Clarity Core as a peer dependency in your libraries to prevent duplication
  of versions in the host application. Peer dependencies is currently the only
  way to manage dependencies with Web Components. This is due to Web Components
  being registered in the global scope. See discussion for Custom Elements Scoped
  Registry support https://github.com/w3c/webcomponents/issues/716
