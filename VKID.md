# Авторизация в web-приложении через VK ID

[VK ID](https://dev.vk.com/vkid/about) — это платформа для авторизации пользователей в сервисах VK. 

Вы можете использовать VK ID для регистрации и авторизации пользователей в своем web-приложении. Для этого необходимо:
1. [Установить VK SDK](#Установка-VK-SDK)
2. [Настроить авторизацию VK ID](#Настройка-VK-ID). 

Воспользуйтесь [демо-стендом](https://demo.vksak.com/#/), чтобы попробовать возможности авторизации через VK ID.
 
## Установка VK SDK  

Установите VK SDK через npm или yarn:

* **npm**

```js
npm install @vkontakte/superappkit
```    
* **yarn**  

```js
add @vkontakte/superappkit
```
    
После установки VK SDK подключите модуль `Connect`:  

```js
import { Connect } from '@vkontakte/superappkit';
```
    
## Настройка VK ID

Далее настройте VK ID. Выберите подходящий способ авторизации и добавьте его в свое приложение. 

Доступные способы:
* [Страница авторизации в текущей вкладке](#страница-авторизации-в-текущей-вкладке)
* [Быстрая авторизация](#Быстрая-авторизация)
* [Авторизация в правом верхнем углу](#Авторизация-в-правом-верхнем-углу)
* [Окно с кнопкой авторизации в любом месте разметки](#Окно-с-кнопкой-авторизации-в-любом-месте)

Дополнительно:
* [Отображение попапа с передаваемыми данными](#Отображение-попапа-с-передаваемыми-данными).

### Страница авторизации в текущей вкладке

```js
Connect.redirectAuth
```    
Это основной способ, при котором VK ID открывает страницу авторизацию в рамках текущей вкладки.

После успешной авторизации пользователя перенаправит на адрес, указанный в параметре `url`.

**Обратите внимание: URL должен иметь протокол HTTPS. При использовании HTTP страница будет бесконечно загружаться.**

В строке запроса URL будет передан параметр `payload` с данными авторизации в формате данных `VKSilentTokenPayload`. Дополнительно можно передать:
* Параметр `state` — любая произвольная строка, которая будет добавлена к url после авторизации. После успешной авторизации в строке запроса URL будет передан параметр `state` имеющий такое же значение, как и переданное в `Connect.redirectAuth`.
* Параметр `screen` со значением `phone`. В этом случае всегда будет отобраться экран с вводом номера телефона, даже если аккаунт пользователя уже найден. 

Пример использования метода:
   
```js
Connect.redirectAuth({ 
  url: 'https://vk.com/',
  state: 'urawesome',
  screen: 'phone'
});
```
### Быстрая авторизация

```js
Connect.oneTapAuth
```   
Это способ быстрой авторизации, при котором веб-приложение распознает, что пользователь уже авторизован в VK, и предлагает быстро авторизоваться под распознанным профилем.

Этот метод принимает два аргумента:
1. `AuthButtonType`, который определяет тип отображения:
	* `floating` — авторизация в виде плавающего окна
	* `button` — авторизация в виде кнопки.
2. `AuthButtonParams` с обязательным полем callback. В него необходимо передать функцию обработчика разных событий `ConnectEvents`, приходящих из SDK.

Пример использования:

```js
Connect.oneTapAuth('floating', {
  callback: () => {
  }
})
```

### Авторизация в правом верхнем углу

```js
Connect.floatingOneTapAuth
```
    
Это аналог метода `Connect.oneTapAuth('floating', ...)`. С его помощью можно поместить окно авторизации в правый верхний угол в виде фиксированного элемента.

Пример использования:
```js
Connect.floatingOneTapAuth({
  callback: function(evt: any) {
    switch (type) {
      case ConnectEvents.OneTapAuthEventsSDK.LOGIN_SUCCESS:
        if (evt.payload.auth) {
          return onAuthUser(evt);
        } else {
          console.error(evt);
        }
        break;
      /**
       * Для событий FULL_AUTH_NEEDED и PHONE_VALIDATION_NEEDED необходимо открыть полноценный VK ID,
       * чтобы пользователь завершил регистрацию или номер подтвердил телефона
       */
      case ConnectEvents.OneTapAuthEventsSDK.FULL_AUTH_NEEDED:
      case ConnectEvents.OneTapAuthEventsSDK.PHONE_VALIDATION_NEEDED:
        return Connect.redirectAuth({
          screen: 'phone',
          url: 'https://vk.com'
        });
    }
  }
})

```    
Чтобы закрыть окно авторизации, используйте метод `VKOneTapAuthResult.destroy`:
```js
  const oneTapObj = Connect.floatingOneTapAuth(...);
    
  ...
    
  oneTapObj.destroy();
```
    
### Окно с кнопкой авторизации в любом месте

```js
Connect.buttonOneTapAuth
```    
Это аналог метода `Connect.oneTapAuth('button', ...)`. С его помощью можно разместить окно с кнопкой авторизации в любом месте разметки. В параметре доступны ещё два свойства:
* `container` — html-элемент, в который будет вставлено окно с кнопкой
* `options` — объект `ButtonOneTapAuthOptions`, который управляет показом элементов внутри окна с кнопкой.  

Пример использования:
```js
const buttonOneTap = Connect.buttonOneTapAuth({
  callback: function(evt: any) {
    const type = evt.type;
    if (!type) {
      return;
    }
    switch (type) {
      ...
      case ConnectEvents.OneTapAuthEventsSDK.LOGIN_SUCCESS:
        return onAuthUser(evt);
      /**
       * Для событий FULL_AUTH_NEEDED и PHONE_VALIDATION_NEEDED необходимо открыть полноценный VK ID,
       * чтобы пользователь завершил регистрацию или номер подтвердил телефона
       */
      case ConnectEvents.OneTapAuthEventsSDK.FULL_AUTH_NEEDED:
      case ConnectEvents.OneTapAuthEventsSDK.PHONE_VALIDATION_NEEDED:
      case ConnectEvents.ButtonOneTapAuthEventsSDK.SHOW_LOGIN:
        return Connect.redirectAuth({
          screen: 'phone',
          url: 'https://...'
        });
      case ConnectEvents.ButtonOneTapAuthEventsSDK.SHOW_LOGIN_OPTIONS:
        /**
         * Параметр screen: 'phone' позволяет сразу открыть окно ввода телефона в VK ID
         */
        return Connect.redirectAuth({
          screen: 'phone',
          source: type
        }).then(onAuthUser, () => alert('Ошибка!'));
    }
    return;
  },
  options: {
    showAlternativeLogin: true,
    showAgreements: false,
    showAgreementsDialog: true,
    displayMode: 'default',
  },
});

if (buttonOneTap) {
  document.getElementById('someHtmlElementId').appendChild(buttonOneTap.getFrame());
}
```    

### Отображение попапа с передаваемыми данными
	
```js
Connect.userDataPolicy
```
    
Этот метод позволяет отображать попап с используемыми политиками и передаваемыми данными. 

Пример использования:
```js
Connect.userDataPolicy(...)
  .show()
  .then(() => {
    console.log('Policy was accepted');
  });
```
## Типы 
### VKSilentAuthPayload
```js
type VKSilentAuthPayload = {
  auth: number;
  token: string;
  ttl: number;
  type: string;
  user: {
    id: number;
    first_name: string;
    last_name: string;
    avatar: string;
    phone: string;
  };
  uuid: string;
  oauthProvider?: string;
  external_user?: ExternalUser;
};
```    
### VKAuthSuccessResult
```js
type VKAuthSuccessResult = {
  provider: 'vk';
  payload: VKSilentAuthPayload;
};
```    
### ExternalUser
```js
type ExternalUser = {
  id?: string;
  avatar?: string;
  first_name: string;
  last_name: string;
  phone: string;
  borderColor?: string;
  payload?: Record<string, any>;
};
```
### ConnectAuthError
```js
type ConnectAuthError = {
  code: ERROR_CODES;
  reason?: string;
};
```

### VKOAuthCallback

```js
interface VKOAuthCallback {
  provider: 'fb' | 'google';
};
```
### RedirectAuthParams
```js
type RedirectAuthParams = {
  url: string;
  state?: string;
  screen?: 'phone';
  action?: AuthAction;
  source?: string;
};
```
### AuthWithUser
```js
interface AuthWithUser {
  name: 'login_with_user';
  token: string;
}
```

### ExtendTokenParams
```js
interface ExtendTokenParams {
  name: 'extend_token';
  token: string;
  params: {
    extend_token_hash: string;
  };
}
```
### AuthAction
```js
type AuthAction = ExtendTokenParams | AuthWithUser;
```
### FloatingOneTapAuthParams
```js
type FloatingOneTapAuthParams = {
 callback: (result: VKAuthButtonCallbackResult) => void;
};
```

### ButtonOneTapAuthParams
```js
interface ButtonOneTapAuthParams extends FloatingOneTapAuthParams {
 container?: HTMLElement | null;
 options?: ButtonOneTapAuthOptions;
}
```
### ButtonOneTapAuthDisplayModes
```js
type ButtonOneTapAuthDisplayModes = 'default' | 'name_phone' | 'phone_name';
```
где:
* `default` — стандартное отображение "Продолжить как ..."
* `name_phone` — отображение в 2 строки:
	* Верхняя строка — "Продолжить как..."
	* Нижняя строка — телефон пользователя.
* `phone_name` — отображение в 2 строки:
	* Верхняя строка — телефон пользователя
	* Нижняя строка — "Продолжить как...".

### ButtonOneTapSkin
```js
type ButtonOneTapSkin = 'primary' | 'flat';
```
### ButtonOneTapAuthOptions
```js
type ButtonOneTapAuthOptions = {
  showAgreements?: boolean | number;
  showAlternativeLogin?: boolean | number;
  showAgreementsDialog?: boolean | number;
  displayMode?: ButtonOneTapAuthDisplayModes;
  buttonSkin?: ButtonOneTapSkin;
  langId?: number;
};
```
где `langId` — это доступные константы языков:
```js
const LANGUAGES = {
  RUS: 0,
  UKR: 1,
  ENG: 3,
  SPA: 4,
  GERMAN: 6,
  POL: 15,
  FRA: 16,
  TURKEY: 82,
};
```

### AuthButtonType
```js
type AuthButtonType = 'floating' | 'button';
```
### AuthButtonParams
```js
type AuthButtonParams = FloatingOneTapAuthParams | ButtonOneTapAuthParams;
```
### VKAuthButtonResult
```js
type VKAuthButtonResult = {
  type: OneTapAuthEventsSDK | FloatingOneTapAuthEventsSDK | ButtonOneTapAuthEventsSDK | DataPolicyEventsSDK;
  provider?: 'vk';
  payload?: VKSilentAuthPayload | VKDataPolicyPayload | { uuid: string };
};
```
### VKAuthButtonResult
```js
type VKAuthButtonError = {
  type: OneTapAuthEventsSDK;
  payload: VKAuthButtonErrorPayload;
};
```
### VKAuthButtonCallbackResult
```js
type VKAuthButtonCallbackResult = VKAuthButtonResult | VKAuthButtonError;
```

### VKOneTapAuthResult
```js
export type VKOneTapAuthResult = {
 destroy: () => void;
 getFrame: () => HTMLIFrameElement | null;
 authReadyPromise: Promise<OneTapAuthEventsSDK>;
};
```

### VKOneTapAuthButtonResult
```js
export type VKOneTapAuthButtonResult = {
 destroy: () => void;
 getFrame: () => HTMLIFrameElement | null;
 authReadyPromise: Promise<OneTapAuthEventsSDK>;
};
```
