# single-spa-portal-example
The goal of this project is to provide an example of how to build a portal like app which consists of multiple single page applications (SPA's). Each SPA should be self contained with its own build process. It should be individually deployable without the need to deploy the whole application if there are changes to any individual app.

This example is based on [simple-single-spa-webpack-example](https://github.com/joeldenning/simple-single-spa-webpack-example/blob/master/README.md) but will provide further features like:

- [x] Isolate SPA's with their own build process
- [x] Load SPA's on demand with SystemJS
- [x] Provide a way to communicate between each SPA
- [x] Get assets (like images, fonts, css, etc.) coming from different servers to work
- [ ] Support multiple Angular versions without zone.js conflicts

## inter-app-communication
This topic has been discussed multiple times (i.e. [here](https://github.com/CanopyTax/single-spa/issues/112) or [here](https://github.com/CanopyTax/single-spa/issues/107)). There may be many solutions to solve this problem. In this repository I want you to show a solution that meets the following requirements:

- Each app is a self contained system. No app knows the internal state of another app or their data model. In short, each app is treated as a black box and can be maintained by a different team. 
- Each app must be able to have a complex state. 
- When you navigate between apps, the state must not be lost (because of mount/unmount).

To meet these requirements I have decided for an event system where each app can or can not listen to events that other apps send. This enables each app to keep their isolated state and modify only their own state based on events from other apps (and probably resend other events). No app needs direct access to the state of another app.

Furthermore I needed to split the apps into two parts. One is the normal app itself (GUI, Framework, etc.), the other is a "communication layer" which is exported as separate module and loaded/instantiated by the portal regardless of the app state. This allows each app to listen and react to events even if they aren't mounted.

Each app can process these events in whatever way they like. The only requirement is that all apps agree on one event format to send and receive these events.

For this example I have decided to just go with redux since it basically does exactly what I need. Throw events and process events. **But this system works with whatever technic you like.**

Here is a graphic which illustrates what actually happens:

![inter-app-communication](docs/inter-app-communication.jpg "inter-app-communication")

### The main parts are:

**StoreApps:** Contains state + business logic. Implements a dispatch method which can be called by the GlobalEventDistributor if an global event occurs.

**GUIApps:** singleSPA middleware + UI code like HTML, CSS, Controller, etc. + Framework like React or Angular

**GlobalEventDistributor:** Can be used to register stores. Sends dispatch events to all stores. (Observer pattern)

### How it works:
1. The root-application boots and loads all stores and instantiates them. This is necessary since we need the communication layer (store) to be active at all times. Even if the whole application is not mounted yet. Otherwise no app specific events getting processed.
2. When an app is being mounted, the root-application passes the already instantiated store belonging to the individual app down. The root-application also passes a reference to the GlobalEventDistributor to the app. 
3. Now you can happily send all your global events to the GlobalEventDistributor with the dispatch() method and all other events to the local store.

### Cons:
As already mentioned, the biggest disadvantage is that all stores have to be loaded when the root-application loads. The reason for this is that we are building a project that will have a huge application state being entirely in the browser. The user will likely input 1h of data without any server communication and once he is done, he will save everything with one click. 
This must not necessarily be your use-case. For example if you are only interested in inter-app-communication with any currently active app, you may not need to load all states beforehand but rather load them while the apps mount.  

## How to use it
1. Clone this project
2. Jump into each folder and do:
   - `npm install`
   - `npm run watch`
4. Open up http://localhost:9000 in a web browser.
