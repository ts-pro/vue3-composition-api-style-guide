# Vue3 Composition Api style guide ( WIP 🚧 )
Vue3 Composition API components &amp; project structure guide.

## Сontents:
- Project
  - Dictionary
  - Directories structure
  - Linter config
- Component
- Module
- Store
- Router

## Project
### Dictionary
- _component_ - *.vue file. Contains html & js defineComponent function. 
- _page_ - component which is referred by router.
- _module_ - js/ts file.
- _hook_ - module which contains parts for component, e.g. refs, computeds, methods, other hooks, DOM interactions.
- _service_ - module which contains third party interaction code or connections to external resources. E.g. vendors initialisation code, rest requests, websocket connections.
- _helper_ - module with exported functions/vars which is not hook or service.

## Component
### Hooks spread
Avoid using hooks result spread in components ( not everywhere but in components ), for some reasons:
1. It requires to count all props in spread and when you return to template, it may be huge list.
2. It increases chances of name collisions ( if 2 hooks export same name ), which can be solved using ugly name aliases.
3. It makes context lost. For example, if you have variable `isVisible` spreaded from some hook, you can't say where it came from until you go deeper. Using more verbose expression `trialPopup.isVisible` it's no uncertain anymore.

Bad: spread
```ts
setup(props) {
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
  } = useUserProfile({
    userId: props.userId,
  });

  init();
  initUserProfile();

  return {
    isReady,
    show,
    hide,
    isReadyUserProfile,
    name,
    country,
    city,
    age,
    isOnline,
    isBlocked,
    isDeleted,
  };
}
```

Good: direct assign
```ts
setup(props) {
  const paymentPopup = usePaymentPopup();
  const userProfile = useUserProfile({ userId: props.userId });

  paymentPopup.init();
  userProfile.init();

  return {
    paymentPopup,
    userProfile,
  };
}
```


### Setup structure
Make it the rule than components can contain template, top level defineComponent props ( e.g. props, name, emit, setup, etc... ) and setup function makes a hooks caller role. Do not write code in setup function except hooks calls and hooks returned method calls. Breaking this rules will make setup function huge, unstructured and unreadable. 

**Good setup structure:**
```ts
setup(props, { emit }) {
  const trial = useTrial(emit);
  const startPack = useStartPack();
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

    // Make something important, which is based on trial.init promise state.
    startPack.setPromoInCache();
  });

  // Return all data from hooks which is needed in template. 
  // Don't return hooks data if you have no plans for use them in template !
  return {
    trial,
    startPack,
    swiper,
  };
}
```


### Async setup
Do not use async setup because it will cause you to use [suspense](https://v3.vuejs.org/guide/migration/suspense.html#introduction) which is still experimental. We offer you a simple pattern for loading async data ( maybe from server ):

Bad: async setup
```ts
// Async requires you to use suspence now...
async setup() {
  const myProfile = useMyProfile();
  await myProfile.load();
  
  // Other code
}
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
setup() {
  // Get my profile 
  const myProfile = useMyProfile();

  // Run profile loading from server.
  myProfile.load();
  
  return {
    // If you don't want to write long accessor like 
    // {{ myProfile.profile.name }} in <template>, you can alias 
    // nested data. Typecast here is required to exclude undefined 
    // from ref for code autocompletion if you are usign Idea WebStorm
    profile: myProfile.profile as Ref<MyProfile>,
  }
}
```

## Module
WIP 🚧

## Store
WIP 🚧

## Router
WIP 🚧
