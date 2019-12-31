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
    <div class="capacitor-only">
      <q-btn color="primary" label="Get Picture" @click="captureImage" />
      <img :src="imageSrc" width="50" height="50">
      <br><br>
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
  </q-page>
</template>

<script>
import { Plugins, CameraResultType, StatusBarStyle } from '@capacitor/core'

const { Geolocation, Camera, Device, Toast, StatusBar, SplashScreen, Share, LocalNotifications } = Plugins
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
        this.latitude = position.coords.latitude
        this.longitude = position.coords.longitude
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
        quality: 90,
        allowEditing: true,
        resultType: CameraResultType.Uri
      })
      // image.webPath will contain a path that can be set as an image src.
      // You can access the original file using image.path, which can be
      // passed to the Filesystem API to read the raw data of the image,
      // if desired (or pass resultType: CameraResultType.Base64 to getPhoto)
      this.imageSrc = image.webPath
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
