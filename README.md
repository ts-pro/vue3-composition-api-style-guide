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
### Setup structure
Make it the rule than components can contain template, top level defineComponent props ( e.g. props, name, emit, setup, etc... ) and setup function makes a hooks caller role. Do not write code in setup function except hooks calls and hooks returned method calls. Breaking this rules will make setup function huge, unstructured and unreadable. 

**Good setup structure:**
```ts
setup(props, { emit }) {
  // Do not spread data from hooks in components, because it's a place
  // where many hooks appear, and there is high risk of name collision.
  // For example `init` method is inside useTrial & useSwiper hooks.
  // And using name aliases like const { init: as initSwiper } = useSwiper();
  // is dirty because now you can have aliased and non-aliased data mixed.
  // Also spread pushes you to return all data you need inside template,
  // it's also make template readability harder, for example, if you have
  // variable {{ profile.name }} in template it's not obvious where it came from.
  // Much easier to export the whole hook and use it more verbose:
  // {{ trial.profile.name }}
  const trial = useTrial(emit);
  // Some useful fature.
  const startPack = useStartPack();
  // Apply swiper.js to dom.
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

```ts
// useMyProfile.ts hook.
export function useMyProfile() {
  // Ref is undefined by default, means than data is not loaded yet.
  const myProfile = ref<MyProfile>();
  
  // Load data from server
  loadMyProfile().then((myProfileValue) => {
    // Set this data when it's already loaded.
    myProfile.value = myProfileValue;
  })
  
  return {
    myProfile,
  }
}
```

```ts
// MyProfile Component.
setup() {
  // Get my profile ref with undefined which will be 
  // defined whenever data is loaded.
  const { myProfile } = useMyProfile();
  
  return {
    // Cast myProfile to it's type to exclude undefine 
    // & enable code completion in template ( it's necessary 
    // for idea webstorm ).
    myProfile: myProfile as Ref<MyProfile>,
  }
}
```

## Module
WIP 🚧

## Store
WIP 🚧

## Router
WIP 🚧
