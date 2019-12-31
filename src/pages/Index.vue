<template>
  <q-page>
    <div>
      GPS position: <strong>{{ latitude }} , {{ longitude }}</strong>
    </div>
    <br><br>
    <div>
      <div>Model: {{ model }}</div>
      <div>Manufacturer: {{ manufacturer }}</div>
    </div>
    <br><br>
      <q-btn color="primary" label="Get Picture" @click="captureImage" />
      <br><br>
      <img :src="imageSrc" v-if="imageSrc" width="50" height="50">
      <br><br>
    <div class="capacitor-only">
      <q-btn color="primary" @click="show_toast" label="Show Toast" />
      <br><br>
      <q-btn color="primary" @click="showStatusBar" label="Show Status" />
      <br><br>
      <q-btn color="primary" @click="hideStatusBar" label="Hide Status" />
      <br><br>
      <q-btn color="primary" @click="changeStatusBar" label="Change Status" />
      <br><br>
      <q-btn @click="share" label="Share" />
      <br><br>
    </div>
      <q-btn @click="local_notification" label="Local Notification" />
      <br><br>
      <q-btn color="negative" @click="showAlert" label="Alert" />
      <q-btn color="primary" @click="showConfirm" label="Confirm" />
      <q-btn color="info" @click="showPrompt" label="Prompt" />
      <q-btn color="warning" @click="showActions" label="Actions" />
  </q-page>
</template>

<script>
import { Plugins, CameraResultType, CameraSource, StatusBarStyle } from '@capacitor/core'

import { defineCustomElements } from '@ionic/pwa-elements/loader'

const { Geolocation, Camera, Device, Toast, Modals, StatusBar, SplashScreen, Share, LocalNotifications, PushNotifications } = Plugins
export default {
  name: 'PageIndex',
  data () {
    return {
      position: 'fetching...',
      latitude: 0.00,
      longitude: 0.00,
      imageSrc: '',
      model: 'loading...',
      manufacturer: 'loading...',
      isStatusBarLight: true
    }
  },
  methods: {
    async showAlert () {
      let alertRet = await Modals.alert({
        title: 'Stop',
        message: 'this is an error'
      })
      console.log('Confirm ret', alertRet)
    },
    async showConfirm () {
      let confirmRet = await Modals.confirm({
        title: 'Confirm',
        message: 'Are you sure you\'d like to press the red button?'
      })
      console.log('Confirm ret', confirmRet)
    },
    async showPrompt () {
      let promptRet = await Modals.prompt({
        title: 'Hello',
        message: 'What\'s your name?'
      })
      console.log('Prompt ret', promptRet)
    },
    async showActions () {
      let promptRet = await Modals.showActions({
        title: 'Photo Options',
        message: 'Select an option to perform',
        options: [
          {
            title: 'Upload'
          },
          {
            title: 'Share'
          },
          {
            title: 'Remove'
          }
        ]
      })
      console.log('You selected', promptRet)
    },
    local_notification () {
      LocalNotifications.schedule({
        notifications: [
          {
            title: 'Local Notification',
            body: 'this is a local notification triggered',
            id: 1,
            schedule: { at: new Date(Date.now() + 1000 * 5) },
            sound: 'res://platform_default',
            attachments: null,
            actionTypeId: '',
            extra: null
          }
        ]
      })
    },
    getCurrentPosition () {
      Geolocation.getCurrentPosition().then(position => {
        this.position = position
        this.latitude = position ? position.coords.latitude : null
        this.longitude = position ? position.coords.longitude : null
      })
    },
    async share () {
      await Share.share({
        title: 'See cool stuff',
        text: 'Really awesome thing you need to see right meow',
        url: 'http://ionicframework.com/',
        dialogTitle: 'Share with buddies'
      })
    },
    async captureImage () {
      const image = await Camera.getPhoto({
        quality: 100,
        allowEditing: false,
        resultType: CameraResultType.Uri,
        source: CameraSource.Camera
      }).catch(err => alert(err))
      // image.webPath will contain a path that can be set as an image src.
      // You can access the original file using image.path, which can be
      // passed to the Filesystem API to read the raw data of the image,
      // if desired (or pass resultType: CameraResultType.Base64 to getPhoto)
      this.imageSrc = image ? image.webPath : null
    },
    async show_toast () {
      await Toast.show({
        text: 'Hello this is a toast..!'
      })
    },
    changeStatusBar () {
      StatusBar.setStyle({
        style: this.isStatusBarLight ? StatusBarStyle.Dark : StatusBarStyle.Light
      })
      this.isStatusBarLight = !this.isStatusBarLight
    },
    hideStatusBar () {
      StatusBar.hide()
    },
    showStatusBar () {
      StatusBar.show()
    }
  },
  mounted () {
    // PWA Elements
    defineCustomElements(window) // Camera implementation for web

    // Push Notifications
    // Register with Apple / Google to receive push via APNS/FCM
    PushNotifications.register()

    // On success, we should be able to receive notifications
    PushNotifications.addListener('registration',
      (token) => {
        alert('Push registration success, token: ' + token.value)
      }
    )

    // Some issue with our setup and push will not work
    PushNotifications.addListener('registrationError',
      (error) => {
        alert('Error on registration: ' + JSON.stringify(error))
      }
    )

    // Show us the notification payload if the app is open on our device
    PushNotifications.addListener('pushNotificationReceived',
      (notification) => {
        alert('Push received: ' + JSON.stringify(notification))
      }
    )

    // Method called when tapping on a notification
    PushNotifications.addListener('pushNotificationActionPerformed',
      (notification) => {
        alert('Push action performed: ' + JSON.stringify(notification))
      }
    )

    // Ask Permissions for notification on web
    Notification.requestPermission().then(function (permission) {
      console.log('Permission Granted')
    })

    this.getCurrentPosition()
    // we start listening
    this.geoId = Geolocation.watchPosition({}, (position, err) => {
      this.position = position
      this.latitude = position.coords.latitude
      this.longitude = position.coords.longitude
    })

    Device.getInfo().then(info => {
      this.model = info.model
      this.manufacturer = info.manufacturer
    })

    // Show the splash for two seconds and then auto hide:
    SplashScreen.show({
      showDuration: 3000,
      autoHide: true
    })
  },
  beforeDestroy () {
    // we do cleanup
    Geolocation.clearWatch(this.geoId)
  }
}
</script>
