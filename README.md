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
- _hook_ - module which contains parts for component, r.g. refs, computeds, methods, other hooks.
- _service_ - module which contains third party interaction code or connections to external resources. E.g. vendors initialisation code, rest requests, websocket connections.
- _helper_ - module with exported functions/vars which is not hook or service.

## Component
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
