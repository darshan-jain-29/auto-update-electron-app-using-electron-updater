Please follow the steps to achieve the electron js auto-update code working.

1. `npm install electron-updater` (For using auto-updater)
2. `npm install electron-log` (This will help to see the errors in logs)
3. `npm install electron --save-dev`
4. `npm install electron-builder --save-dev`

Then, in your main.js (the entry file of your application paste auto update code at as mentioned)

    const electron = require('electron');
    const url = require('url');
    const path = require('path');
    const pjson = require('../package.json')
    
    // auto update code
    const log = require('electron-log');
    const { autoUpdater } = require("electron-updater");
    autoUpdater.logger = log;
    log.info('App starting...');
    let appversion = pjson.version;
    
    const { app, BrowserWindow } = electron;
    
    let homePageWindow;
    
    function sendStatusToWindow(text, ver) {
        log.info(text);
        homePageWindow.webContents.send('message', text, ver);
    }
    
    function createDefaultWindow() {
        homePageWindow = new BrowserWindow({
            minimizable: true,
            maximizable: true,
            closable: true,
        });
        homePageWindow.maximize();
        homePageWindow.webContents.openDevTools();
        //homePageWindow.setMenu(null);
    
        homePageWindow.on('closed', () => {
            homePageWindow = null;
            app.quit();
        });
    	
        homePageWindow.loadURL(url.format({
            pathname: path.join(__dirname, '../webFiles/homePage.html'),
            protocol: 'file:',
            slashes: true
     

       }));
        return homePageWindow;
    }
    
    // auto update conditions
    autoUpdater.on('checking-for-update', () => {
        sendStatusToWindow('Checking for update...', appversion);
    })
    
    autoUpdater.on('update-available', (info) => {
        sendStatusToWindow('Update available.', appversion);
    })
    
    autoUpdater.on('update-not-available', (info) => {
        sendStatusToWindow('Update not available.', appversion);
    })
    
    autoUpdater.on('error', (err) => {
        sendStatusToWindow('Error in auto-updater. ' + err, appversion);
    })
    
    autoUpdater.on('download-progress', (progressObj) => {
        let log_message = "Download speed: " + progressObj.bytesPerSecond;
        log_message = log_message + ' - Downloaded ' + progressObj.percent + '%';
        log_message = log_message + ' (' + progressObj.transferred + "/" + progressObj.total + ')';
        sendStatusToWindow(log_message, appversion);
    })
    
    autoUpdater.on('update-downloaded', (info) => {
        setTimeout(function () {
            sendStatusToWindow('Update downloaded..Restarting App in 5 seconds', appversion);
            homePageWindow.webContents.send('updateReady');
            autoUpdater.quitAndInstall();
        }, 5000)
    });
    
    app.on('ready', function () {
        createDefaultWindow();
        autoUpdater.checkForUpdatesAndNotify();
    });
    
    app.on('window-all-closed', () => {
        app.quit();
    });

Refer to my code and do the needful changes in your main.js

Then set the version of your application to say 1.0.0
Run command "npm run build --win -p always" to generate first build 

I use Github to store the apps components.
So, you need to upload the app files of 1.0.0 (latest.yml, builder-effective-config.yaml, app_setup1.0.0.exe, app_setup1.0.0.exe.blockmap file) on git and create a release for it. Publish the release as 1.0.0

Then change the version of the app to 1.0.1 
Run command "npm run build --win -p always" to generate second build.

Again do the github step - you need to upload the app files of 1.0.1 (latest.yml, builder-effective-config.yaml, app_setup1.0.1.exe, app_setup1.0.1.exe.blockmap file) on git and create a release for it. Publish the release as 1.0.1

So now in your project on git you have 2 releases 1.0.0 and 1.0.1

Now for testing the things you need to run the setup of 1.0.0 on your local machine. So, if the code is written correctly you will see logs that 
1. Update available
2. Update downloaded and its %

Then once 100% download is completed the app gets restarted in 5 seconds (as per seconds mentioned in the code) and your app is updated now.
Yayyyyyyyyy
