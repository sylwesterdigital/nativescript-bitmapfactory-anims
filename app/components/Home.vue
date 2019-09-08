<template>
  <Page class="page">
    <GridLayout columns="*,*" rows="*,40,40">
      <Image ref="myImage" :src="imgSrc" stretch="none" row="0" col="0" rowSpan="3" colSpan="2" @touch="onTouch" @tap="onTap" />
<!--      <Button row="1" col="0" @tap="playDemo" :text="playButtonText"></Button>
      <Button row="1" col="1" @tap="shareDemo" :text="shareButtonText"></Button>
      <Slider class="gravitySlider" row="2" col="0" colSpan="2" :value="grav" @valueChange="onSliderGravityChange"/>-->
    </GridLayout>
  </Page>
</template>

<script>
  const platformModule = require("tns-core-modules/platform");
  
  
import { fromUrl, ImageSource } from "image-source";
import * as SocialShare from "nativescript-social-share";
  
/*  const SocialShare = require("nativescript-social-share");
  const ImageSourceModule = require("image-source"); */
  
  import BitmapFactory from "nativescript-bitmap-factory";
  import KnownColors from "color/known-colors";
  
  
//  import Image from "tns-core-modules/ui/image";
//  import fromBase64 from "image-source";
  
  
/*
  import {
    Color
  } from "tns-core-modules/color"
*/
  
  //const bmp = BitmapFactory.create(640, 480);
  
  let scale = 1;
  let width = platformModule.screen.mainScreen.widthDIPs*scale;
  let height = (platformModule.screen.mainScreen.heightDIPs)*scale;
  
  console.log(width,height)
  
  let numBalls = 30;
  let spring = 0.3;
  let gravity = 0.04;
  let friction = -0.3;
  let balls = [];
  let W = width;
  let H = height;
  let MW = Math.floor(W / 2)
  let MH = Math.floor(H / 2)
  let maxDiameter = 45;
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
        // console.log(others[i]);
        let dx = this.others[i].x - this.x;
        let dy = this.others[i].y - this.y;
        let distance = sqrt(dx * dx + dy * dy);
        let minDist = this.others[i].diameter + this.diameter;
        //   console.log(distance);
        //console.log(minDist);
        if (distance < minDist) {
          //console.log("2");
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
      //this.fill = colorsArr[this.id]
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
        grav: ((gravity*100)+100)/2,
        msg: "Hello!",
        imgSrc: "",
        mytime: 0,
        demoState: "Stop",
        playButtonText: "Play",
        shareButtonText: "Share",        
        touchPoint: [50, 50],
        newHEX: '#' + Math.random().toString(16).slice(2, 8)
      }
    },
    
    computed: {
      message() {
        return "Blank {N}-Vue app";
      }
    },
    
    mounted() {
      console.log('mounted');
      this.$nextTick(function() {
          //console.log('entire view has been rendered');
        }),
        this.interval = setInterval(() => {
        
        
        
          this.updateTimer();
        }, 10);
      this.updateTimer();
      
      this.setupBalls()
      
      this.playDemo();
    },
    
    
    methods: {
      
      onSliderGravityChange(e) {
        gravity = -1 + (e.value*0.02);
      },
      
      shareDemo() {
        this.demoState = "Stop";
        let image = this.imgSrc;
        SocialShare.shareImage(image);
      },
      
      setupBalls() {
        
        console.log('setupBalls')
        
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
      
      onTap() {
        this.newHEX = this.getNewHex();
        
        //gravity = Number((Math.random()*-1 + Math.random()*1).toFixed(3))
        
        this.grav = ((gravity*100)+100)/2
        
      },
      
      onTouch(args) {
        this.touchPoint = [parseInt(args.getX()), parseInt(args.getY())];
        //console.log("Touch ",this.touchPoint[0]);
        //          this.currentTouch = "Touch: x: " + parseInt(args.getX()) + " y: " + parseInt(args.getY())+" action: "+args.action;
        //
        //          if(args.action == 'up') {
        //          }
        //
        //          if(args.action == 'move') {
        //          }
        //
                  if(args.action == 'down') {
                    let perc = parseInt(args.getY())/H
                    console.log('down',perc)
                    gravity = Number(((2*perc) - 1).toFixed(3));
                    this.grav = ((gravity*100)+100)/2
                  }
      },
      
      updateTimer() {
        
        //this.grav = gravity;

        
        if (this.demoState == "Play") {
          this.mytime += 0.005;
          
          let bmp = BitmapFactory.create(width, height);
          var path = Number(Math.sin(this.mytime) * 0.95).toFixed(3);
          
          bmp.dispose(() => {
            
/*            balls.forEach(ball => {
              ball.collide();
              ball.move();
              //let coords = ball.x+","+ball.y;
              bmp.drawCircle(ball.diameter, ball.x+","+ball.y, ball.col);
            });*/            
          
            
            for(let a in balls) {
              const r = balls[a];
              r.collide()
              r.move()
              bmp.drawCircle(r.diameter, r.x+","+r.y, r.col);
            }
            
            
            let coords = this.touchPoint.toString();
            let hex = this.newHEX;
            
//            let i = 0;
//            while (i < 8) {
//              i++;
//              bmp.drawCircle(20 + path * i*20, coords, hex);
//            }
            
//            bmp.writeText("PATH: " + path, "16,16", {
//              color: "#0000FF",
//              size: 16,
//              name: "Avenir"
//            });
//            
//            bmp.writeText("HEX: " + this.newHEX, "16,41", {
//              color: this.newHEX,
//              size: 16,
//              name: "Avenir"
//            });
//            
            bmp.writeText("G: " + gravity, "160,10", {
              color: "#FF69B4",
              size: 20,
              name: "Avenir"
            });            
            
            //var data = bmp.toDataUrl(BitmapFactory.OutputFormat.JPEG, 75);
            //var base64JPEG = bmp.toBase64(BitmapFactory.OutputFormat.JPEG, 75);
            
            this.imgSrc = bmp.toImageSource();
            //this.imgSrc = bmp.toDataUrl(BitmapFactory.OutputFormat.JPG, 75);
            //this.imgSrc = fromBase64(bmp.toBase64(BitmapFactory.OutputFormat.JPEG, 75));
            
            
          });
          
          //bmp.dispose(function() {});
          bmp = null;
        }
      },
      
      playDemo() {
        
        console.log('playDemo() - this.demoState:', this.demoState)
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
        font-family: "Avenir";
		padding: 12;
		background: white;
		margin: 4;
		border-radius: 8;
		border-width: 2;
		border-color: pink;
	}
  
  
</style>