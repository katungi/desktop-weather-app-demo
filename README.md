### Getting Started with Electron with Create React App

Creating Desktop applications has come a long way. With every passing day, developers come up with easy to use options for creating desktop applications. Electron is one of those solutions. It uses web technologies wrapped around Node.js to come up with web technologies. For a more detailed introduction to the whole Electron.Js ecosystem, you can read [this article](https://www.section.io/engineering-education/cross-platform-applications-electron/).

Electron uses HTML/CSS traditionally. However, you can use React and other JavaScript frameworks with electron. In this article, we will:

- Create a React.js app using [Create-React-App](https://reactjs.org/docs/create-a-new-react-app.html)
- Install Electron into the application
- Configure Electron in the React.js app
- Finally, create a demo desktop application using Electron and React.

### Prerequisites

To follow along comfortably with this tutorial, The reader must:

- Have basic Knowledge of React and how it works
- Have Node.js installed. If not, they can install it from [here](https://nodejs.org/en/download/)
- Have read the introduction article on Electron on the Section Blog [here](https://www.section.io/engineering-education/cross-platform-applications-electron/)

### Setup The Application

Now that we know our goal and how to get there. Let's get started. There are many ways to create a React app. For the purposes of Simplicity, we will use Create-React-App (CRA) which is a project generator for React apps. It is important to also note that, when building a React app with CRA, it is relatively easier to create desktop applications compared to when you are building the React project from scratch.

With that said, create a folder or navigate into your preferred location and name it anything. Then run CRA using npx as shown below. I will name my application electron-react-demo.

```terminal
cd ~/ your-prefered-location

npx create-react-app electron-react-demo
```

The npx command will create a React app called electron-react-demo.
When that command is finished, we navigate into the directory and install electron. We do this in the terminal like this:

```terminal
cd electron-react-demo

npm i -D electron
```

We also installed a useful npm package called `electron-is-dev` for checking whether our electron app is in development or production.

The `-D` is not necessary, we just used the flag so that we can install electron under dev dependancies. Depending on the strength of your internet, this command might take a while.

The next step is to create a configuration file for Electron. Normally you would create it in the root of the project, However in React we will create it in the public folder where all the HTML code is located. so we create a file in the `public` called electron.js `/public/electron.js`.

The next step is adding the Electron Boilerplate. It is the same code you'd write in a plain HTML/CSS electron app, just with a few changes. Paste this code into the `electron.js` file:

```js
const path = require('path');

const { app, BrowserWindow } = require('electron');
const isDev = require('electron-is-dev');

function createWindow() {
  // Create the browser window.
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    },
  });

  // and load the index.html of the app.
  // win.loadFile("index.html");
  win.loadURL(
    isDev
      ? 'http://localhost:3000'
      : `file://${path.join(__dirname, '../build/index.html')}`
  );
  // Open the DevTools.
  if (isDev) {
    win.webContents.openDevTools({ mode: 'detach' });
  }
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(createWindow);

// Quit when all windows are closed, except on macOS. There, it's common
// for applications and their menu bar to stay active until the user quits
// explicitly with Cmd + Q.
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow();
  }
});
```

The code basically creates a Browserwindow, then loads the HTML file in the directory. It also handles when you close the window and other window operations. To read more on the boilerplate, you can visit the [docs](https://www.electronjs.org/docs/tutorial/quick-start).

The significant change is that we added the a custom html file to be launched. It will be in out build file, which will be the destination in production.

#### Configuring package.json

Now we have electron installed, but we have to make a few changes in the package.json to syncronize the browser and desktop builds. First we update the entry file for the project. In your package.json, add this to the top level scripts:

```json
  "main" : "main": "public/electron.js",
```

Next, we want to listen to the app such that, when it lauches on the browser, we intercept that and launch it as an electron app instead. To do this, we need some help from npm packages. We need `concurrent` and `wait-on`.

```terminal
npm i -D concurrently wait-on
```

Concurently allows us to run mutliple commands within one script and wait-on will wait for port 3000 which is the default CRA port, to launch the app. We will also pass a flag `BROWSER=none` in one of the scripts to prevent the browser from launching once out React app compiles successfully. So now under `scripts` in our `package.json` file, we add:

```json
...
    "dev": "concurrently -k \"BROWSER=none npm start\" \"npm:electron\"",
    "electron": "wait-on tcp:3000 && electron .",
```

And just like that, we have everything set up. Running `npm run dev` should launch an electron app.

### Demo Time

Now that we have everything set up, let's try and build a simple web application using react, and see it become a desktop application.

We will make a weather application using react. To get started, head over to the [open weather website](https://openweathermap.org/api) and create an account. Once you are signed in, You will be directed to a dashboard.

Navigate to `api-keys` tab and enter the name of an api key in the input provided. I named mine `react demo` and click the `generate` button to create a new api-key for our application. We are doing this to get weather data from the API and feed it into our app.

![dashboard](/dashboard.png)

Now that we have our API-Key, let's start building. Npm has a handy package for building out weather components for react applications, its called [react-open-weather](https://www.npmjs.com/package/react-open-weather). It will generate the User interface we want. To install it, in the terminal, type:

```terminal
npm i react-open-weather
```

When that is installed, Let's go ahead and add the component into our react app.

Navigate into your `src` folder and into the `app.js` file, that is where all our work will be.

First things first, let's import the component from the react-open-weather package. We will import the component and import a helper function for using the open weather api, which is provided by the package.

```js
import ReactWeather, { useOpenWeather } from 'react-open-weather';
```

The import takes place at the very top of our application. Within the App function, let's use our `useOpenWeather` function to bring in data from the api. We do so by specifying a few necessary things like longitude and latitude, the unit of metrics and also passing your api key.

```js
const { data, isLoading, errorMessage } = useOpenWeather({
  key: 'you api key',
  lat: '1.2921',
  lon: '36.8219',
  lang: 'en',
  unit: 'metric', // values are (metric, standard, imperial)
});
```

We are going to hard code the latitude and longitude values, but you can make them dynamic to give a better feel.

Finally, in our return statement for the App function, we return a beautiful weather component with the data from the API. We will also pass it some information as props. Some of this information includes, unit labels for the data, language, we will ofcourse pass in the data. The code will look like this:

```js
<div className='App'>
  <ReactWeather
    isLoading={isLoading}
    errorMessage={errorMessage}
    data={data}
    lang='en'
    locationLabel='Nairobi'
    unitsLabels={{ temperature: 'C', windSpeed: 'Km/h' }}
    showForecast
  />
</div>
```

With that, our weather application is complete. The full code for the app.js file should look like this:

```js
import ReactWeather, { useOpenWeather } from 'react-open-weather';
import './App.css';

function App() {
  const { data, isLoading, errorMessage } = useOpenWeather({
    key: 'you api key',
    lat: '1.2921',
    lon: '36.8219',
    lang: 'en',
    unit: 'metric', // values are (metric, standard, imperial)
  });

  return (
    <div className='App'>
      <ReactWeather
        isLoading={isLoading}
        errorMessage={errorMessage}
        data={data}
        lang='en'
        locationLabel='Nairobi'
        unitsLabels={{ temperature: 'C', windSpeed: 'Km/h' }}
        showForecast
      />
    </div>
  );
}

export default App;
```

You can now launch your application as a desktop application by running `npm run dev`.

It should look like this:

![dashboard](/electron-demo-result.png)

## Conclusion

In this article, we covered how to setup a simple desktop application using Create-React-App and Electron. We also created a simple weather application to demonstrate how to everything works together.

The whole premise is that Electron provides your web application with a low-level bindings. It wraps your application around a BrowserWindow and launches it as a desktop application.

Electron comes with all the power of Node.JS so you can basically do anything Node can do in Electron. Now that you have learned all this, you can now convert your existing React projects into desktop applications or build cool desktop projects.

You can find the full code for the tutorial [here](https://github.com/katungi/desktop-weather-app-demo)

Happy Coding!
