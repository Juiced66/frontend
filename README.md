# Interactive POI (Point of Interests) Map

In this challenge you will create an interactive map that displays markers representing Points of Interests.

Estimated time:

- Junior: 4 hours
- Middle: 3 hours
- Senior: 2 hours
- Liar: 1 hour

Technologies:

- Kuzzle Backend  ✔️
- Kuzzle JS SDK ✔️
- Vue.js ✔️
- Typescript

## Goals

- Understand client need from basic spec
- Initialize Typescript project with separate backend and frontend 
- Ability to find information in documentation 
- Fetch data from Kuzzle API using the SDK
- Manage asynchronous and realtime flux of data
- Integrate a Map with Leaflet

Some key words that interests us: asynchronous, spinner, error handling, KISS, UI/UX, easter egg

## Description

_The description is intentionally vague._

Upon arrival of the user on the application, a map is displayed.

A click on the map opens a modal that asks for the name of the POI to add, the POI is then saved.

POIs are initially colored in yellow to indicate that you want to visit them.

Clicking on a yellow POI changes its color to green to indicate that it has been visited.

A click on a green POI deletes it.

BONUS: The map should be synchronized in real time using Kuzzle's realtime engine. This means that if I open two tabs and add a POI from one tab, it appears in the other tab simultaneously.

## Resources

_It's up to you to look deep into the documentation_

- [Run Kuzzle Backend](https://docs.kuzzle.io/core/2/guides/getting-started/run-kuzzle/)
- [Data Storage Guide](https://docs.kuzzle.io/core/2/guides/main-concepts/data-storage/)
- [Realtime Guide](https://docs.kuzzle.io/core/2/guides/main-concepts/realtime-engine/)
- [Javascript SDK](https://docs.kuzzle.io/sdk/js/7/)
- [Vue Leaflet (Map)](https://github.com/vue-leaflet/vue-leaflet)

## Capture
![Capture](https://user-images.githubusercontent.com/80784430/199967687-18b30a3f-c9b4-4295-a733-3421fd202bb5.PNG)

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
