# Vue3 Composition Api style guide ( WIP 🚧 )
Vue3 Composition API components &amp; project structure guide.

## Сontents:
- Project
  - Tech  
  - Dictionary
  - Directories structure
  - Linter config
- Component
- Module
- Store
- Router

## Project
### JavaScript or TypeScript ?
Aou are feel free to choose. We use TS because it makes code strict, safe and more autocomplete friendly. But it is not necessary to use TS for reading this guide, examples are compatible with JS, there are bit TS syntax can be met.

### setup() or <script setup> ?
If you are using build tools ( vite, webpack, rollup, etc... ) it's better to use <script setup> notation, because it offers you many [advantages](https://vuejs.org/guide/introduction.html#composition-api). But here we cover styles trying not to be tightly coupled to this style, you can use components with manually written setup() as well.

### Dictionary
- _component_ - *.vue file. Contains html & js defineComponent function. 
- _page_ - component which is referred by router.
- _module_ - js/ts file.
- _hook_ - module which contains parts for component, e.g. refs, computeds, methods, other hooks, DOM interactions.
- _service_ - module which contains third party interaction code or connections to external resources. E.g. vendors initialisation code, rest requests, websocket connections.
- _helper_ - module with exported functions/vars which is not hook or service.

### Directories structure
Basic example or project directory structure
```ts
|--src
    |--assets // All media content.
    |   |--images // Images or svg.
    |   |--styles // css/scss.
    |   |--audio // mp3, wav, ogg, ...
    |
    |--components // Components than can be shared between pages.
    |   |--Cart // All components should have own folder named in StudlyCaps notation.
    |       |--Cart.vue // Cart component, can be used or multiple pages, that's why it's in src/components
    |       |--__tests__ // All unit-tests are near their testing targets.
    |           |--Cart.test.ts // Test file has name equals to target, but with suffix `.test`.
    |
    |--helpers
    |   |--scroll // Each helper is in own folder.
    |       |--scroll.ts
    |       |--__tests__
    |           |--scroll.test.ts
    |
    |--hooks
    |   |--use-model // Each hook is in own folder.
    |     |--use-model.ts
    |     |--__tests__
    |         |--use-model.test.ts
    |
    |--services
    |   |--google-maps // Each service is in own folder.
    |       |--google-maps.ts
    |           |--__tests__
    |               |--google-maps.test.ts
    |
    |--pages // Contains components which is targeted by router, same structure rules.
        |--components
            |--CategoryList
                |--CategoryList.vue // This components is available by route /category/list.
                |--components // All local componens are next to parent.
                   |--ListItem
                   |   |--ListItem.vue // This component is used only in parent components, that's why it's here.
                   |
                   |--ListFilter
                       |--ListFilter.vue // Some other local component for CategoryList.
```
## Component
### Hooks spread
Avoid using hooks result spread in components if it's not necessary. There are cases when you shouldn't use spread:
1. If it requires to count all props in spread, it may be huge list.
2. If it leads to name collisions ( if 2 hooks export same name ), which can be solved using ugly name aliases.
3. If it makes context lost in template. For example, if you have variable `isVisible` spreaded from some hook, you can't say where it came from until you go deeper. Use more verbose expression `trialPopup.isVisible`, it's no uncertain anymore.

Bad: useless spread usage
```ts
const { init, isReady, show, hide } = usePaymentPopup();
const {
  init: initUserProfile,
  isReady: isReadyUserProfile ,
  name,
  country,
  city,
  age,
  isOnline,
  isBlocked,
  isDeleted,
} = useUserProfile({ userId });

// Bad: Two different name styles to avoid name collisions ( spreaded & aliased ).
init();
initUserProfile();
```

Good: direct assign
```ts
const paymentPopup = usePaymentPopup();
const userProfile = useUserProfile({ userId });

paymentPopup.init();
userProfile.init();
```

### Setup structure
Make it the rule that components can contain template, and some top-level data ( props, emits, hooks usage). Do not write code in components except hooks calls and hooks returned method calls. Breaking this rules will make components, unstructured and unreadable. 

**Bad setup**
```ts
// Bad: props are defined in setup
const price = ref(0);
const isAvailable = ref(false);
const swiper = new Swiper();

// Bad: all logics are here using refs above, which leads to huge all-in-one components.
```

**Good: no refs creation, logics use variables from hooks**
> It's a complex example of interactions & dependencies between hooks 
```ts
const emit = defineEmits(['close']);

// Passing emit in hook is okay, if it fires events from inside.
const trial = useTrial(emit);
const swiper = useSwiper();

// Some logics can be applied only after data loading
trial.init().then(() => {
  // Now it's possible to start initializing swiper
  swiper.init();

  // We have some logics based on 2 hooks, but admit that we do not define
  // new refs or functions, we just use data from hooks.
  if (startPack.getPromoCache().count > 0) {
    swiper.value.slideTo(4);
  }
});

```
> Tip: you can define function in component if there is a function to connect logics between hooks, e.g.:
```ts
const emit = defineEmits(['close']);

const trial = useTrial(emit);
const nav = useNav();
// Or use spread here: const { closeModal } = useNav();

// It's okay to define method to use data from both hooks: useTrial & useNav.
function buyTrial() {
  // Here you can use async/await as well.
  trial.startTransaction().then((isSuccess) => {
    if (isSuccess) {
      nav.goPaymentSuccessPage();
    } else {
      nav.goPaymentFailPage();
    }
  });
}
```
But it's just an example of allowed functions to define. Of course it would be okay to move `this` logics under other hook to encapsulate all logics from component, and leave just data which is required for usage in template. Above component followed this rule:
```ts
const emit = defineEmits(['close']);

// buyTrial now is inside of useTrialActions hook.
const { buyTrial } = useTrialActions(emit);
```

### Async setup
Do not use async setup because it will require you to use [suspense](https://v3.vuejs.org/guide/migration/suspense.html#introduction) which is still experimental. We offer you a simple pattern for loading async data ( maybe from server ):

Bad: async setup
```ts
// Await requires you to use suspence now...
const myProfile = useMyProfile();
await myProfile.load();
  
// Other code
```

Good: normal setup
```ts
// useMyProfile.ts hook.
export function useMyProfile() {
  // Ref is undefined by default, means than data is not loaded yet.
  const profile = ref<MyProfile>();
  
  function load(): void {
    // Load data from server.
    return loadMyProfile().then((myProfileValue) => {
      // Set this data when it's already loaded.
      profile.value = myProfileValue;
    })
  }
  
  return {
    load,
    profile,
  };
}
```
```vue
<template>
  <!-- Required check if profile is loaded ( is not undefined ) -->
  <div v-if="profile">
    {{ profile.name }}
  </div>
  <!-- v-else you can show any loading spinner -->
</template>
```
```ts
// MyProfile Component.
const myProfile = useMyProfile();

// Run profile loading from server.
myProfile.load();
```

## Module
WIP 🚧

## Store
WIP 🚧

## Router
WIP 🚧
