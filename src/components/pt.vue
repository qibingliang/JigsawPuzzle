<template>
    <div id="JigsawPuzzle" style="width:100%;height:500px;background-color: #9ac2b9;">
        <el-upload
            class="avatar-uploader"
            action="https://jsonplaceholder.typicode.com/posts/"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <img v-if="imageUrl" id="imgpt" :src="imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
        行：<el-input-number
            v-model="valueY"
            controls-position="right"
            :min="2"
            :max="10"
            label="描述文字"
        ></el-input-number>
        列：<el-input-number
            v-model="valueX"
            controls-position="right"
            :min="2"
            :max="10"
            label="描述文字"
        ></el-input-number><br/>
        <el-button type="primary" @click="clickCropImage()">生成</el-button>
        <el-button type="primary" @click="Disruption()">打乱</el-button>
        <div>
            <transition-group name="flip-list" tag="div">
            <template v-for="(itemY, y) in locationArr">
                <template v-for="(item, x) in itemY">
                    <img 
                        @click="move(item, y, x)" 
                        :key="uniqueId+item.index" 
                        :style="item.index == numberX * numberY ? 'opacity: 0;' : 'cursor: pointer;'"
                        style="vertical-align: bottom;margin:2px;"
                        :src="item.src"
                    />
                    <!-- <br v-if="x == numberX - 1" :key="`br_${item.index}`"/> -->
                </template>
                <br :key="`${uniqueId}_br_${y}`"/>
            </template>
            </transition-group>
        </div>
        <div style="position:fixed;bottom: -2000px;">
            <!-- <img id="imgpt1" style="width: 100px;height: 100px;z-index:-100;" src="./wallhaven-686616.jpg"/> -->
            <canvas id="imageC" style="z-index:-100;" width="200" height="200"></canvas>
        </div>
    </div>
</template>

<script>
export default {
    name: "",
    components: {},
    props: {},
    data() {
        return {
            valueX: 4,
            numberX: 4,
            valueY: 4,
            numberY: 4,
            locationArr: [],
            imageUrl: '/img/wallhaven-686616.9cdc7c84.jpg',
            uniqueId:0,
            imgShow:false,
        };
    },
    computed: {},
    watch: {},
    created() {},
    mounted() {},
    methods: {
        //打乱拼图
        Disruption(){
            let arr = this.$lodash.flatten(this.locationArr);
            arr.sort((a, b)=>this.randomsort(a, b));
            // console.log('%c专属log：','color:#468cd4;font-size:12px;',arr);
            this.locationArr.forEach((el,i)=>{
                el.forEach((el2,i2)=>{
                    el2=arr[this.numberX*i+i2]
                    this.$set(this.locationArr[i],i2,arr[this.numberX*i+i2])
                })
            });
            this.$forceUpdate();
            // console.log('%c专属log：','color:#468cd4;font-size:12px;',this.locationArr);
        },
        randomsort(a, b) {
            return Math.random()>.5 ? -1 : 1;
            //用Math.random()函数生成0~1之间的随机数与0.5比较，返回-1或1
        },
        //点击移动
        move(item, y, x) {
            // console.log("%c当前点击块：","color:#468cd4;font-size:12px;","y:"+y,"x:"+x,item);
            if (item.index !== this.numberX * this.numberY) {
                let x9, y9;
                this.locationArr.forEach((el, index) => {
                    el.forEach((el2, index2) => {
                        if (el2.index == this.numberX * this.numberY) {
                            y9 = index;
                            x9 = index2;
                        }
                    });
                });
                // console.log("%c空格位置：","color:#468cd4;font-size:12px;","y:" + y9,"x:" + x9);
                if ( (x == x9 && Math.abs(y - y9) === 1) || (y == y9 && Math.abs(x - x9) === 1) ) {
                    [this.locationArr[y][x], this.locationArr[y9][x9]] = [this.locationArr[y9][x9], this.locationArr[y][x]];
                    this.$forceUpdate();
                }
                // console.log("%clocationArr：","color:#468cd4;font-size:12px;",this.locationArr);
            }
        },
        //分割图片
        async clickCropImage() {
            this.uniqueId = this.$lodash.uniqueId('id_');
            this.createImg();
            this.locationArr = [];
            this.numberX = this.valueX;
            this.numberY = this.valueY;
            this.imgShow = false;
            await this.$nextTick();
            let canvas = document.getElementById("imageC"); //  获取画布大小，判断画布大小
            let canvasH = canvas.clientHeight;
            let canvasW = canvas.clientWidth; //  将图片等分
            let splaceX = 0;
            let splaceY = 0;
            for (let y = 0; y < this.numberY; y++) {
                let list = [];
                for (let x = 0; x < this.numberX; x++) {
                    let location = {
                        x: (x * canvasW) / this.numberX - splaceX,
                        y: (y * canvasH) / this.numberY - splaceY,
                        Dx: (x * canvasW) / this.numberX + canvasW / this.numberX,
                        Dy: (y * canvasH) / this.numberY + canvasH / this.numberY,
                        locationOption: (x + 1).toString() + (y + 1).toString()
                    };
                    let src = this.cropImage(
                        canvas,
                        (x * canvasW) / this.numberX - splaceX,
                        (y * canvasH) / this.numberY - splaceY,
                        canvasW / this.numberX,
                        canvasH / this.numberY
                    );
                    list.push({ src, index: this.numberX * y + x + 1 });
                }
                this.locationArr.push(list);
            }
            await this.$nextTick();
            this.imgShow = true;
            // canvasComimgCreated = false;
            // divComimgCreated = false;
        },
        cropImage(targetCanvas, x, y, width, height) {
            let targetctx = targetCanvas.getContext("2d");
            let targetctxImageData = targetctx.getImageData(x,y,width,height); // sx, sy, sWidth, sHeight
            let c = document.createElement("canvas");
            let ctx = c.getContext("2d");
            c.width = width;
            c.height = height;
            ctx.rect(0, 0, width, height);
            ctx.fillStyle = "white";
            ctx.fill();
            ctx.putImageData(targetctxImageData, 0, 0); // imageData, dx, dy // 创建img 块
            return c.toDataURL("image/jpeg", 0.92);
        },
        createImg() {
            let canvas = document.getElementById("imageC");
            let ctx = canvas.getContext("2d");
            let img = document.getElementById("imgpt");
            ctx.drawImage(img, 0, 0, 200, 200);
        },
        handleAvatarSuccess(res, file) {
            console.log('%c专属log：','color:#468cd4;font-size:12px;',file);
            this.imageUrl = URL.createObjectURL(file.raw);
        },
        beforeAvatarUpload(file) {
            const isJPG = file.type === 'image/jpeg';
            const isLt2M = file.size / 1024 / 1024 < 10;
            if (!isJPG) {
                this.$message.error('选择图片只能是 JPG 格式!');
            }
            if (!isLt2M) {
                this.$message.error('选择图片大小不能超过 10MB!');
            }
            if(isJPG && isLt2M){
                this.imageUrl = URL.createObjectURL(file);
            }
            return false;
            // return isJPG && isLt2M;
        }
    },
};
</script>

<style lang="scss" scoped>
.flip-list-move {
  transition: transform 0.3s;
}
.flip-list-enter, .flip-list-leave-to {
  opacity: 0;
}
#JigsawPuzzle {
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }
    .el-upload--text {
        width: auto;
        height: auto;
    }
    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 178px;
        height: 178px;
        line-height: 178px;
        text-align: center;
        margin-top: 0px;
    }
    .avatar {
        width: 178px;
        height: 178px;
        display: block;
    }
}
</style>
