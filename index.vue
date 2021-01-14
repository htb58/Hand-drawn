<template>
  <div class="add-sign" v-if="signShow">
    <div class="sign-conter">
      <div class="close">
        <Icon type="md-close" @click="singClose"/>
      </div>
      <header>
        <div>
          手绘签名
        </div>
      </header>
      <main>
        <div id="add_signature">
          <div class="title">
            提示：请在以下区域绘制签名！
          </div>
          <canvas
            id="canvas"
            width="895"
            height="350"
            ref="canvas"
          >
          抱歉，您的浏览器不支持canvas元素
          </canvas>
          <div class="choose-color">
              <div v-for="(v,i) in borderColor" @click="choiceColor(v.tag)" :style="{borderColor:v.color}" :key="i" :class="{active: color == v.tag}">
                <span :style="{background:v.color}"></span>
              </div>
          </div>
          <div class="btn">
            <Button @click="clear">
              清空
            </Button>
            <Button type="primary" :loading="signloading" @click="getDrawPic">
              提交
            </Button>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>
<script>
import SignaturePad from 'signature_pad'
export default {
  name: 'addsignature',
  props:{
    signShow: {
      type: Boolean,
      default: false,
      required: true
    },
    type: {
      type: String,
      default: 'all'
    }
  },
  data() {
    return {
      color:1,
      ctx: {},
      canvas: {},
      canvasMoveUse: false,
      number:0,
      signaturePad:null,
      penColor: '#000',
      borderColor:[
        {color:'#000',tag:1},
        {color:'red',tag:0},
        {color:'#0C77FA',tag:2}
      ],
      signloading: false
    }
  },
  watch:{
    'signShow'(n , o){
      if(n){
        this.init();
        this.params.flag = 1;
        this.getDefaultMoudle();
        this.active = 1;
        this.color = '1';
      }
    },
    "color"(n){
      switch(n){
        case '1':
          this.penColor = '#000';
          break;
        case '0':
          this.penColor = 'red';
          break;
        case '2':
          this.penColor = '#0C77FA';
          break;  
      }
    },
  },
  computed:{
    username () {
      return this.$store.state.userInfo.userInfo.username
    }
  },
  methods:{
    singClose(params){
      this.$emit('closeAddsign',params)
    },
    init() {
      this.$nextTick(()=>{
        this.canvas = this.$refs.canvas;
        this.signaturePad  = new SignaturePad(this.canvas,{
          penColor: this.penColor,
          lineWidth: 5,
          minWidth: 1.5
        }); 
      })
    },
    getDrawPic() {
      this.signloading = !this.signloading
      if (this.signaturePad.isEmpty()){
        this.$Message.error('签章不能为空');
        return;
      }
      var data = this.signaturePad.toDataURL("image/png");
      this.img = data;
      var data = this.cropSignatureCanvas(this.canvas);
      if (data) {
        var dataURL_ok = data.replace(/data:image\/.*;base64,/,'')
        var url = this.type == 'shouhui'? 'drawsealDefault' : 'drawpersonalseal'
        return this.$http.post(url, {base64: dataURL_ok}).then(res => {
          this.signloading = !this.signloading
          if(res && res.data && res.data.status === this.GlobalStatus.SUCCESS){
            this.$Message.success('签章上传成功')
            this.clear()
            this.singClose('success')
          }else{
            this.$Message.error('签章上传失败')
          }
        })
        this.$emit("drawPicSuccess");
      } else {
        this.$Message.error('签章不能为空')
      }
    },
    clear(){
      this.signaturePad.clear();
    },
    cropSignatureCanvas: function(canvas) {
      // 复制画布 避免影响之前的画布
      var croppedCanvas = document.createElement('canvas'),
          croppedCtx    = croppedCanvas.getContext('2d');

          croppedCanvas.width  = canvas.width;
          croppedCanvas.height = canvas.height;
          croppedCtx.drawImage(canvas, 0, 0);

        // 处理画布空白
        var w         = croppedCanvas.width,
            h         = croppedCanvas.height,
            pix       = {x:[], y:[]},
            imageData = croppedCtx.getImageData(0,0,croppedCanvas.width,croppedCanvas.height),
            x, y, index;
        for (y = 0; y < h; y++) {
            for (x = 0; x < w; x++) {
                index = (y * w + x) * 4;
                if (imageData.data[index+3] > 0) {
                    pix.x.push(x);
                    pix.y.push(y);
                }
            }
        }
         
        pix.x.sort(function(a,b){return a-b});
        pix.y.sort(function(a,b){return a-b});
        var n = pix.x.length-1;

        w = pix.x[n] - pix.x[0];
        h = pix.y[n] - pix.y[0];
        var cut = croppedCtx.getImageData(pix.x[0], pix.y[0], w, h);

        croppedCanvas.width = w;
        croppedCanvas.height = h;
        croppedCtx.putImageData(cut, 0, 0);

        return croppedCanvas.toDataURL("image/png");
    },
    choiceColor(i,type){
      this.color = i+'';
      this.clear();
      this.signaturePad.off()
      this.signaturePad = null;
      this.init();
    }
  }
}
</script>
<style lang="less">
  .add-sign{
    position: fixed;
    left: 0;
    top:0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background-color: rgba(0,0,0,0.6);
    z-index: 999;
    .sign-conter{
      position: absolute;
      left: 0;
      right: 0;
      top:0;
      bottom: 0;
      margin: auto;
      width: 955px;
      height: 623px;
      z-index: 66;
      background: #fff;
      border-radius: 6px;
      padding: 30px;
      padding-top: 15px; 
      .close{
        height: 22px;
        text-align: right;
        font-size: 22px;
        line-height: 18px;
        i{
          cursor: pointer;
        }
      }
      header{
        display: flex;
        justify-content:center;
        color: #3F4558;
        font-size: 16px;
        font-weight: bold;
        div{
          height: 30px;
          margin-right: 40px;
          cursor: pointer;
        }
        div.active{
          color: #4680FF;
          border-bottom: 2px solid #4680FF;
        }
      }
      main{
        #add_signature {
          width: 100%;
          margin: 0 auto;
          #canvas{
            width: 100%;
            height: 350px;
            border: 1px solid #dedede;
          }
          .title{
            font-size: 14px;
            color:#ccc;
            line-height: 50px;
            height: 50px;
          }
          
        }
        .sign-moudle{
          margin-top: 30px;
          .sign-moudle-body{
            display: flex;
            margin-bottom: 30px;
          }
          .show-signature{
            width: 350px;
            height: 350px;
            border: 1px solid #eee;
            margin-right: 40px;
            header{
              padding: 14px;
              text-align: center;
              font-size: 14px;
              color: #c3c4ca;
            }
            section{
              width: 100%;
              height: 240px;
              text-align: center;
              display: flex;
              align-items: center;
              justify-content: center;
              img{
                display:inline-block;
                width: 148px;
              }
            }
            .choose-color{
              display: flex;
              justify-content: center;
            }
          }
          .sign-moudle-list{
            width: calc(100% - 390px);
            .title{
              font-size: 14px;
              color: #3F4558;
              margin-bottom: 14px;
            }
            .body{
              ul{
                height: auto;
                display: flex;
                flex-wrap: wrap;
                li{
                  width: 148px;
                  height: 148px;
                  margin-right:20px;
                  margin-bottom: 20px; 
                  border: 1px solid #eee;
                  padding: 30px;
                  img{
                    width: 100%;
                  }
                }
                li.active{
                  border-color:#FF6619; 
                }
              }
            }
          }
        }
        .choose-color{
          padding: 20px 0;
          display: flex;
          div{
            width: 22px;
            height: 22px;
            line-height: 22px;
            border-radius: 50%;
            margin-right: 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            span{
              display: inline-block;
              width: 16px;
              height: 16px;
              border-radius: 50%;
            }
          }
          div.active{
            border-width: 1px;
            border-style: solid;
          }
        }
        .btn{
          margin: 0 auto;
          text-align: center;
          button{
            width: 100px;
            height: 36px;
            cursor: pointer;
            font-size: 14px;
          }
        }
      }
      .sign-close{
        width: 60px;
        height: 60px;
        border-radius: 50%;
        background: #fff;
        position: relative;
        top:100px;
        margin: 0 auto;
      }
    }
    
  }
</style>