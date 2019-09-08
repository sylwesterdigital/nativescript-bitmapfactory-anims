<template>
  <Page class="page">
    <GridLayout columns="*,*" rows="40,*,40,40">
      <Image ref="myImage" :src="imgSrc" stretch="none" row="1" col="0" rowSpan="3" colSpan="2" @touch="onTouch" @tap="onTap" />
      <Button row="2" col="0" @tap="playDemo" :text="demoState"></Button>
      <Button row="2" col="1" @tap="shareDemo" :text="shareButtonText"></Button>
      <Slider class="gravitySlider" row="3" col="0" colSpan="2" :value="grav" @valueChange="onSliderGravityChange"/>
      <Label :text="fps" row="0" col="0"></Label>
      <Label :text="'Gravity: '+calcGrav().toFixed(3)" row="0" col="1"></Label>
    </GridLayout>
  </Page>
</template>

<script>
  
  const fpsMeter = require("tns-core-modules/fps-meter");
  const platformModule = require("tns-core-modules/platform");
  import * as SocialShare from "nativescript-social-share";
  import BitmapFactory from "nativescript-bitmap-factory";
  
  let scale = 1;
  let width = platformModule.screen.mainScreen.widthDIPs*scale;
  let height = (platformModule.screen.mainScreen.heightDIPs)*scale;
  let numBalls = 25;
  let spring = 0.3;
  let gravity = -0.008;
  let friction = -0.3;
  let balls = [];
  let W = width;
  let H = height;
  let MW = Math.floor(W / 2)
  let MH = Math.floor(H / 2)
  let maxDiameter = 35;
  let colorsArr = [];
  
  
  class Ball {
    constructor(xin, yin, din, idin, oin, col, id) {
      this.x = xin;
      this.y = yin;
      this.vx = 0;
      this.vy = 0;
      this.diameter = din;
      this.id = idin;
      this.others = oin;
      this.col = col;
      this.id = id;
    }
    collide() {
      for (let i = this.id + 1; i < numBalls; i++) {
        let dx = this.others[i].x - this.x;
        let dy = this.others[i].y - this.y;
        let distance = sqrt(dx * dx + dy * dy);
        let minDist = this.others[i].diameter + this.diameter;
        if (distance < minDist) {
          let angle = atan2(dy, dx);
          let targetX = this.x + cos(angle) * minDist;
          let targetY = this.y + sin(angle) * minDist;
          let ax = (targetX - this.others[i].x) * spring;
          let ay = (targetY - this.others[i].y) * spring;
          this.vx -= ax;
          this.vy -= ay;
          this.others[i].vx += ax;
          this.others[i].vy += ay;
        }
      }
    }
    move() {
      this.vy += gravity;
      this.x += this.vx;
      this.y += this.vy;
      if (this.x + this.diameter / 2 > width) {
        this.x = width - this.diameter / 2;
        this.vx *= friction;
      } else if (this.x - this.diameter / 2 < 0) {
        this.x = this.diameter / 2;
        this.vx *= friction;
      }
      if (this.y + this.diameter / 2 > height) {
        this.y = height - this.diameter / 2;
        this.vy *= friction;
      } else if (this.y - this.diameter / 2 < 0) {
        this.y = this.diameter / 2;
        this.vy *= friction;
      }
    }
  }
  
  
  
  export default {
    data() {
      return {
        fps: "N/A",
        grav: this.calcGrav(),
        imgSrc: null,
        mytime: 0,
        demoState: "Stop",
        playButtonText: "Play",
        shareButtonText: "Share",        
        touchPoint: [MW, MH],
        newHEX: '#' + Math.random().toString(16).slice(2, 8)
      }
    },
    computed: {
      message() {
        return "Computed";
      }
    },
    mounted() {
      this.$nextTick(function() {
        }),
        this.interval = setInterval(() => {
          this.updateTimer();
        }, 10);
      this.updateTimer();
      this.setupBalls();
      this.playDemo();
      this.startFPSmeter();
    },
    methods: {
      
      startFPSmeter() {
        let callbackId = fpsMeter.addCallback((fps, minFps) => {
                //vm.set("fps", fps.toFixed(2));
                //vm.set("minfps", minFps.toFixed(2));
          //console.log(fps.toFixed(2),minFps.toFixed(2))
          this.fps = "FPS: "+fps.toFixed(2)+"/"+minFps.toFixed(2);
        });

        fpsMeter.start();        
        
      },
      onSliderGravityChange(e) {
        gravity = -1 + (e.value*0.02);
        this.grav = this.calcGrav();
      },
      shareDemo() {
        this.demoState = "Stop";
        let image = this.imgSrc;
        SocialShare.shareImage(image);
      },
      setupBalls() {
        for (let i = 0; i < numBalls; i++) {
          colorsArr.push(this.getNewHex())
          balls[i] = new Ball(
            Math.floor(Math.random()*width),
            Math.floor(Math.random()*height),
            Math.ceil(Math.random()*maxDiameter),
            i,
            balls,
            colorsArr[i],
            i+1
          );
        }
      },
      
      getNewHex() {
        return '#' + Math.random().toString(16).slice(2, 8);
      },
      
      calcGrav() {
        return ((gravity*100)+100)/2
      },
      
      onTap() {
        this.newHEX = this.getNewHex();
        this.grav = this.calcGrav();
      },
      
      onTouch(args) {
        this.touchPoint = [parseInt(args.getX()), parseInt(args.getY())];
        if(args.action == 'down') {
          let perc = parseInt(args.getY())/H
          console.log('down',perc)
          gravity = Number(((2*perc) - 1).toFixed(3));
          this.grav = ((gravity*100)+100)/2
        }
      },
      
      updateTimer() {
        if (this.demoState == "Play") {
          
          this.mytime += 0.005;
          
          let bmp = BitmapFactory.create(width, height);
          var path = Number(Math.sin(this.mytime) * 0.95).toFixed(3);
          
          bmp.dispose(() => {
            for(let a in balls) {
              const r = balls[a];
              r.collide()
              r.move()
              bmp.drawCircle(r.diameter, r.x+","+r.y, r.col);
            }
            let coords = this.touchPoint.toString();
            let hex = this.newHEX;
            
/*            bmp.writeText("G: " + gravity, "160,10", {
              color: "#FF69B4",
              size: 20,
              name: "Avenir"
            });*/            
            //var data = bmp.toDataUrl(BitmapFactory.OutputFormat.JPEG, 75);
            //var base64JPEG = bmp.toBase64(BitmapFactory.OutputFormat.JPEG, 75);
            this.imgSrc = bmp.toImageSource();
            //this.imgSrc = bmp.toDataUrl(BitmapFactory.OutputFormat.JPG, 75);
            //this.imgSrc = fromBase64(bmp.toBase64(BitmapFactory.OutputFormat.JPEG, 75));
          });
          
          bmp.dispose(function() {});
          bmp = null;
          
        }
      },
      
      playDemo() {
        this.playButtonText = this.demoState;
        if (this.demoState != "Play") {
          this.demoState = "Play";
        } else {
          this.demoState = "Stop";
        }
      }
      
    }
  };
</script>



<style scoped lang="scss">
  
  @import '../app-variables';
  
	.gravitySlider {
		color: deeppink;
		width: 80%;
		text-align: center;
		background: black;
		transform: scale(0.6)
	}
  
  	Button {
		color: deeppink;
		font-size: 18;
/*
        font-family: "Avenir";
*/
		padding: 12;
		background: white;
		margin: 4;
		border-radius: 8;
		border-width: 2;
		border-color: pink;
	}
  
  Label {
    font-size: 12;
    color:dodgerblue;
    padding: 12;
  }
  
</style>