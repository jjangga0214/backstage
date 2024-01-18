---
id: built-in-extensions
title: App Built-in Extensions
sidebar_label: Built-in extensions
# prettier-ignore
description: Configuring or overriding built-in extensions
---

Built-in extensions are default app extensions that are always installed when you create an Backstage app.

## Disable built-in extensions

All built-in extensions can be disabled in the same way as you disable any other extension:

```yaml title="app-config.yaml"
extensions:
  # Disabling the built-in app root alert element
  - app-root-element:app/alert-display: false
```

:::warning
Be careful when disabling built-in extensions, as there may be other extensions depending on their existence. For example, the built-in "alert display" extension displays messages retrieved via [AlertApi](https://backstage.io/docs/reference/core-plugin-api.alertapi) and disabling this extension will cause the application to no longer display these messages unless you install another extension that displays messages from `AlertApi`.
:::

## Override built-in extensions

You can override any built-in extension whenever their customizations, whether through configuration or input, do not meet a use case for your Backstage instance. Check out [this](../architecture/05-extension-overrides.md) documentation on how to override application extensions.

:::warning
Be aware there could be some implementation requirements to properly override an built-in extension, such as using same apis and do not remove inputs or configurations otherwise you can cause a side effect in other parts of the system that expects same minimal behavior.
:::

## Default built-in extensions

### App

This extension is the first extension attached to the extension tree. It is responsible for receiving the application's root element and other Frontend framework inputs.

#### Inputs

| Name         | Description                | Type                                                                                                                                                   | Requirements | Optional | Default                                                    | Extension creator                                                                                                |
| ------------ | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------ | -------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| root         | The app root element.      | [coreExtensionData.reactElement](https://backstage.io/docs/reference/frontend-plugin-api.coreextensiondata)                                            | -            | false    | Element outputed by the [`App/Root`](#app-root) extension. | Override the `App/Root` extension, or customize it.                                                              |
| apis         | The app apis factories.    | [createApiExtension.factoryDataRef](https://backstage.io/docs/reference/frontend-plugin-api.createapiextension.factorydataref)                         | -            | false    | See [default apis](#default-apis-extensions).              | [createApiExtension](https://backstage.io/docs/reference/frontend-plugin-api.createapiextension)                 |
| themes       | The app themes list.       | [createThemeExtension.themeDataRef](https://backstage.io/docs/reference/frontend-plugin-api.createthemeextension.themedataref)                         | -            | false    | See [default themes](#default-theme-extensions).           | [createThemeExtension](https://backstage.io/docs/reference/frontend-plugin-api.createthemeextension)             |
| components   | The app components list.   | [createComponentExtension.componentDataRef](https://backstage.io/docs/reference/frontend-plugin-api.createcomponentextension.componentdataref)         | -            | false    | See [default components](#defaults-components-extensions). | [createComponentExtension](https://backstage.io/docs/reference/frontend-plugin-api.createcomponentextension)     |
| translations | The app translations list. | [createTranslationExtension.translationDataRef](https://backstage.io/docs/reference/frontend-plugin-api.createtranslationextension.translationdataref) | -            | false    | -                                                          | [createTranslationExtension](https://backstage.io/docs/reference/frontend-plugin-api.createtranslationextension) |

#### Default theme extensions

Extensions that provides a default `light` and `dark` app themes.

| kind  | namespace | name  |        id         |
| :---: | :-------: | :---: | :---------------: |
| theme |    app    | light | `theme:app/light` |
| theme |    app    | dark  | `theme:app/dark`  |

#### Default components extensions

Extensions that provides default components for core app component refs.

|    kind    | namespace |                 name                  |           id           |
| :--------: | :-------: | :-----------------------------------: | :--------------------: |
| components |    app    |       core.components.progress        | `components:app/light` |
| components |    app    |   core.components.notFoundErrorPage   | `components:app/dark`  |
| components |    app    | core.components.errorBoundaryFallback | `components:app/dark`  |

#### Default apis extensions

Extensions that provides a default factory for the core app apis.

// TODO

### App root

This is the extension that creates the app root element, so it renders root level components such as app router, layout.

#### Inputs

| Name       | Description                                                                             | Requirements                                                                                                                                                               | Optional | Default                                                                                                                                                                                      | Extension creator                                                                                                       |
| ---------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| router     | A React component that should manager the app routes context.                           | It must be one [router](https://reactrouter.com/en/main/routers/picking-a-router#web-projects) component or a custom component compatible with the 'react-router' library. | true     | [BrowserRouter](https://reactrouter.com/en/main/router-components/browser-router)                                                                                                            | createRouterExtension                                                                                                   |
| signInPage | A React component that should render the app sign-in page.                              | Should call the `onSignInSuccess` prop when the user has been successfully authorized, otherwise the user will not be correctly redirected to the application home page.   | true     | The default `AppRoot` extension does not use a default component for this input, it bypasses the user authentication check and always renders all routes when a login page is not installed. | createSignInPageExtension                                                                                               |
| children   | A React component that renders the app sidebar and main content in a particular layout. | -                                                                                                                                                                          | false    | The `AppLayout` extension output component, see here.                                                                                                                                        | No creator available.                                                                                                   |
| elements   | React elements to be redered outside of the app layout, such as shared popups.          | -                                                                                                                                                                          | false    | There are two built-in elements that are always installed in the app: An [alert display](#alert-display) an [oauth request dialog](#oauth-request-dialog).                                   | [createAppRootElementExtension](https://backstage.io/docs/reference/frontend-plugin-api.createapprootelementextension/) |
| wrappers   | React components that should wrap the root element.                                     | -                                                                                                                                                                          | true     | createAppRootWrapperElementExtension                                                                                                                                                         |

#### Alert Display

An app root element extension that displays messages posted via the [`AlertApi`](https://backstage.io/docs/reference/core-plugin-api.alertapi) is the documentation for using this api in your components).

|       kind       | namespace |     name      |                  id                  |
| :--------------: | :-------: | :-----------: | :----------------------------------: |
| app-root-element |    app    | alert-display | `app-root-element:app/alert-display` |

##### Configurations

| Key                | Type                                                                       | Default value                             | Description                                                      |
| ------------------ | -------------------------------------------------------------------------- | ----------------------------------------- | ---------------------------------------------------------------- |
| transientTimeoutMs | number                                                                     | 5000                                      | Time in miliseconds to wait before displaying messages           |
| anchorOrigin       | { vertical: 'top' \| 'bottom', horizontal: 'left' \| 'center' \| 'right' } | { vertical: 'top', horizontal: 'center' } | Position on the screen where the message alert will be displayed |

##### Override or disable the extension

If you do not want to display alerts, disable this extension or if the available settings do not meet your needs, override this extension.

:::warning
The built-in "alert display" extension displays messages retrieved via [AlertApi](https://backstage.io/docs/reference/core-plugin-api.alertapi) and disabling this extension will cause the application to no longer display these messages unless you install another extension that displays messages from `AlertApi`.
:::

#### OAuth Request Dialog

An app root element extension that renders the oauth request dialog, it is based on the [oauthRequestApi](https://backstage.io/docs/reference/core-plugin-api.oauthrequestapi/).

|       kind       | namespace |         name         |                     id                      |
| :--------------: | :-------: | :------------------: | :-----------------------------------------: |
| app-root-element |    app    | oauth-request-dialog | `app-root-element:app/oauth-request-dialog` |

### App layout

Renders the app's sidebar and main content in a specific layout.

| kind | namespace |  name  |      id      |
| :--: | :-------: | :----: | :----------: |
|  -   |    app    | layout | `app/layout` |

#### Inputs

| Name    | Description                                        | Type                           | Requirements                             | Optional | Default | Extension creator                                 |
| ------- | -------------------------------------------------- | ------------------------------ | ---------------------------------------- | -------- | ------- | ------------------------------------------------- |
| nav     | A React element that renders the app sidebar.      | coreExtensionData.reactElement | See `App/Routes` overrides requirements. | false    | -       | No creators, override the `App/Routes` extension. |
| content | A React element that renders the app main content. | coreExtensionData.reactElement | See `App/Nav` overrides requirements.    | false    | -       | No creators, override the `App/Nav` extension.    |

### App nav

This extension is responsible for rendering the logo and items in the app's sidebar.

| kind | namespace | name |    id     |
| :--: | :-------: | :--: | :-------: |
|  -   |    app    | nav  | `app/nav` |

#### Inputs

| Name  | Description               | Type                                                                                                                      | Requirements | Optional | Default | Extension creator                                                                                        |
| ----- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------ | -------- | ------- | -------------------------------------------------------------------------------------------------------- |
| logos | A nav logos object.       | [logoElementsDataRef](https://backstage.io/docs/reference/frontend-plugin-api.createnavlogoextension.logoelementsdataref) | -            | true     | -       | [createNavLogoExtension](https://backstage.io/docs/reference/frontend-plugin-api.createnavlogoextension) |
| items | Nav items target objects. | [targetDataRef](https://backstage.io/docs/reference/frontend-plugin-api.createnavitemextension.targetdataref)             | -            | true     | -       | [createNavItemExtension](https://backstage.io/docs/reference/frontend-plugin-api.createnavitemextension) |

### App routes

Renders a route element for each route received as input and a `NotFoundErrorPage` component.

| kind | namespace |  name  |      id      |
| :--: | :-------: | :----: | :----------: |
|  -   |    app    | routes | `app/routes` |

#### Inputs

| Name   | Description        | Type                        | Requirements                                                                                                         | Optional | Default             | Extension creator |
| ------ | ------------------ | --------------------------- | -------------------------------------------------------------------------------------------------------------------- | -------- | ------------------- | ----------------- |
| routes | Routes definition. | createPageExtension.dataRef | Should define routes using the `react-router` library, and get the `NotFoundErrorPage` component via Components API. | -        | createPageExtension |
