# IonicCapExitAlertToast
Alert Box or Toast in App Exit


Open app.component.ts

install cordova plugins
npm install --save @capacitor/core @capacitor/cli


app.component.ts

import { Component, ViewChildren, QueryList, OnDestroy } from '@angular/core';

import { Platform, IonRouterOutlet, AlertController } from '@ionic/angular';
import { SplashScreen } from '@ionic-native/splash-screen/ngx';
import { StatusBar } from '@ionic-native/status-bar/ngx';
import { Router } from '@angular/router';


export class AppComponent implements OnDestroy {

  // for storing the returned subscription
  backButtonSubscription;
  @ViewChildren(IonRouterOutlet) routerOutlets: QueryList<IonRouterOutlet>;

  // set up hardware back button event.
  lastTimeBackPress = 0;
  timePeriodToExit = 2000;
  
    constructor(
    private platform: Platform,
    private splashScreen: SplashScreen,
    public alertController: AlertController,
    private statusBar: StatusBar,
    public router: Router,
  ) {
    this.initializeApp();
  }

  initializeApp() {
    this.platform.ready().then(() => {
      this.statusBar.styleDefault();
      this.backButtonEvent();
      this.splashScreen.hide();
    });
  }
  
  
    backButtonEvent() {
    this.backButtonSubscription = this.platform.backButton.subscribe(async () => {
      this.routerOutlets.forEach((outlet: IonRouterOutlet) => {
        if (outlet && outlet.canGoBack()) {
          outlet.pop();
        } else if (this.router.url === "/home") {
          // this.presentAlertConfirm();
          if (new Date().getTime() - this.lastTimeBackPress < this.timePeriodToExit) {
            navigator['app'].exitApp();
          } else {
            this.showToast('Press back again to exit App.');
            this.lastTimeBackPress = new Date().getTime();
          }
        }
      });
    });
  }


  async showToast(msg) {
    await Toast.show({
      text: msg
    });
  }


  async presentAlertConfirm() {
    const alert = await this.alertController.create({
      header: 'Confirm!',
      message: 'Confirm to Exit App !!!',
      buttons: [
        {
          text: 'Cancel',
          role: 'cancel',
          cssClass: 'secondary',
          handler: (blah) => {
            console.log('Confirm Cancel: blah');
          }
        }, {
          text: 'Exit',
          handler: () => {
            console.log('Confirm Okay');
            navigator["app"].exitApp();
          }
        }
      ]
    });
    await alert.present();
  }






  //Called when view is left
  ngOnDestroy() {
    // Unregister the custom back button action for this page
    this.backButtonSubscription.unsubscribe();
  }

}
