<template>
  <div class="select_invoice">
    <UploadImg />
  </div>
</template>
<script>
const FileUpload = '';
export default {
  name: 'UploadImgTest',
  data() {
    return {
      headerImage:'',
      picValue:'',
    };
  },
  methods: {
    uploadSectionFile(f) { //	附件上传
			let that = this;
			let Orientation;
			let ndata;
			if(f.file.size <= 1 * 1024 * 1024){
        //判断图片是否小于1M,是就直接上传
        ndata=f.file;
        that.postImg(ndata);
			}else{
				//反之压缩图片
        let reader = new FileReader();
				// 将图片2将转成 base64 格式
				reader.readAsDataURL(f.file);
				console.log(reader);
				// 读取成功后的回调
				reader.onloadend = function () {
          let result = this.result;
          let img = new Image();
          img.src = result;
          img.onload = function () {
            let data = that.compress(img,Orientation);
            that.headerImage = data;
            ndata=  that.compress(img,Orientation);
            //BASE64转图片
            var arr = ndata.split(',');
            var mime = arr[0].match(/:(.*?);/)[1];
            var bstr = atob(arr[1]);
            var n = bstr.length;
            var u8arr = new Uint8Array(n);
            while(n--){
              u8arr[n] = bstr.charCodeAt(n);
            }
            ndata=new File([u8arr],f.file.name, {type:mime});
            that.postImg(ndata);
          };
        }; 
      }
    },
    async postImg (ndata) {
			let param = new FormData(); //创建form对象
      param.append('file', ndata); //通过append向form对象添加数据
      let config = {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      };
      try {
        const response = await FileUpload(param, config);  //此处写接口对应的vue方法
        if (response.Code == 200) {
            this.handleFileUploadSuccess(response.Data);
        }
      } catch (error) {
        console.log(error);
      }
    },
    compress(img,Orientation) {
      let canvas = document.createElement('canvas');
      let ctx = canvas.getContext('2d');
        //瓦片canvas
      let tCanvas = document.createElement('canvas');
      let tctx = tCanvas.getContext('2d');
      let initSize = img.src.length;
      let width = img.width;
      let height = img.height;
      //如果图片大于四百万像素，计算压缩比并将大小压至400万以下
      let ratio;
      if ((ratio = width * height / 4000000) > 1) {
        console.log('大于400万像素');
        ratio = Math.sqrt(ratio);
        width /= ratio;
        height /= ratio;
      } else {
        ratio = 1;
      }
      canvas.width = width;
      canvas.height = height;
      //铺底色
      ctx.fillStyle = '#fff';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      //如果图片像素大于100万则使用瓦片绘制
      let count;
      if ((count = width * height / 1000000) > 1) {
        console.log('超过100W像素');
        count = ~~(Math.sqrt(count) + 1); //计算要分成多少块瓦片
        //计算每块瓦片的宽和高
        let nw = ~~(width / count);
        let nh = ~~(height / count);
        tCanvas.width = nw;
        tCanvas.height = nh;
        for (let i = 0; i < count; i++) {
          for (let j = 0; j < count; j++) {
            tctx.drawImage(img, i * nw * ratio, j * nh * ratio, nw * ratio, nh * ratio, 0, 0, nw, nh);
            ctx.drawImage(tCanvas, i * nw, j * nh, nw, nh);
          }
        }
      } else {
        ctx.drawImage(img, 0, 0, width, height);
      }
      //修复ios上传图片的时候 被旋转的问题
      if(Orientation != '' && Orientation != 1){
        switch(Orientation){
          case 6://需要顺时针（向左）90度旋转
            this.rotateImg(img,'left',canvas);
            break;
          case 8://需要逆时针（向右）90度旋转
            this.rotateImg(img,'right',canvas);
            break;
          case 3://需要180度旋转
            this.rotateImg(img,'right',canvas);//转两次
            this.rotateImg(img,'right',canvas);
            break;
        }
      }
      //进行最小压缩
      let ndata = canvas.toDataURL('image/jpeg', 0.1);
      console.log('压缩前：' + initSize);
      console.log('压缩后：' + ndata.length);
      console.log('ndata:' +ndata);
      console.log('压缩率：' + ~~(100 * (initSize - ndata.length) / initSize) + '%');
      tCanvas.width = tCanvas.height = canvas.width = canvas.height = 0;
      return ndata;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
