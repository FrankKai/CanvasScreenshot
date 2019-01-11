<template>
    <div class="picture-container">
        <ul class="picture-list">
            <li class="picture-list-item" v-for="(picture,index) in productPicture.list" :key="index"
                @click="selectPicture(picture)"><img :src="picture"/></li>
        </ul>
        <div class="picture-selected">
            <canvas id="canvas" class="canvas-area" width="260" height="208"></canvas>
            <img id="source" :src="imgClip.src" class="img-source"/>
            <p><span>可上下拖动图片选择封面显示部分，封面分辨率为520px*416px</span></p>
        </div>
    </div>
</template>

<script>
    const generateClip = () => ({
        dWidth: 260,
        dHeight: '',
        downY: 0,
        upY: 0,
        offset: 0,
        canvas: '',
        src: '',
        image: '',
        qiniuImg: '',
    });
    export default {
        name: 'screenshot',
        data() {
            return {
                imgClip: generateClip(),
                productPicture: {
                    list: [
                        'http://pajrnn0dg.bkt.clouddn.com/19-25-50.jpg',
                        'http://pajrnn0dg.bkt.clouddn.com/123.png',
                        'http://pajrnn0dg.bkt.clouddn.com/13-25-50.jpg',
                    ],
                    selected: '',
                },
            };
        },
        mounted() {
            this.generatePicture();
        },
        methods: {
            generatePicture() {
                const canvas = document.getElementById('canvas');
                const ctx = canvas.getContext('2d');
                const image = document.getElementById('source');
                this.imgClip.canvas = canvas;
                this.imgClip.ctx = ctx;
                this.imgClip.image = image;

                this.imgClip.ctx.clearRect(0, 0, 260, 208);

                let lastOffsetY;

                const handleMousemove = (e) => {
                    const currentOffsetY = e.offsetY;
                    this.imgClip.offset = this.imgClip.offset + (currentOffsetY - lastOffsetY);
                    const topBorder = 0;
                    const bottomBorder = -this.imgClip.dHeight + 208;
                    if (this.imgClip.offset < topBorder && this.imgClip.offset > bottomBorder) {
                        this.render(this.imgClip.offset);
                    } else if (this.imgClip.offset < bottomBorder) {
                        this.imgClip.offset = bottomBorder;
                    } else if (this.imgClip.offset > topBorder) {
                        this.imgClip.offset = topBorder;
                    }
                    lastOffsetY = e.offsetY;
                };

                canvas.addEventListener('mousedown', (e) => {
                    // console.log('帅哥按下了鼠标：', 'x:', e.offsetX, 'y:', e.offsetY);
                    lastOffsetY = e.offsetY;
                    this.imgClip.downY = e.offsetY;
                    canvas.addEventListener('mousemove', handleMousemove);
                });

                canvas.addEventListener('mouseup', () => {
                    // console.log('帅哥松开了鼠标：', 'y:', e.offsetY);
                    canvas.removeEventListener('mousemove', handleMousemove);
                });
                canvas.addEventListener('mouseleave', () => {
                    // console.log('帅哥离开了画布：', 'y:', e.offsetY);
                    canvas.removeEventListener('mousemove', handleMousemove);
                });
            },
            selectPicture(value) {
                this.productPicture.selected = value;
                this.imgClip.image.setAttribute('crossOrigin', 'Anonymous');
                this.imgClip.src = this.productPicture.selected;
                const _detail = this;
                this.imgClip.image.onload = () => {
                    const rate = parseFloat((_detail.imgClip.image.width / 260).toFixed(10));
                    _detail.imgClip.dHeight = parseFloat(_detail.imgClip.image.height / rate).toFixed(10);
                    _detail.imgClip.ctx.drawImage(_detail.imgClip.image, 0, 0, _detail.imgClip.dWidth, _detail.imgClip.dHeight);
                };
            },
            render(offset) {
                this.imgClip.ctx.clearRect(0, 0, 260, 208);
                this.imgClip.ctx.drawImage(this.imgClip.image, 0, offset, this.imgClip.dWidth, this.imgClip.dHeight);
            },
        },
    };
</script>

<style lang="scss" scoped>
    .picture-container {
        .picture-selected {
            margin-bottom: 10px;
            overflow: hidden;
            img {
                width: 100%;
            }
            .canvas-area {
                border: 1px dashed #ccc;
                cursor: grab;
            }
            .img-source {
                display: none;
            }
        }
        .picture-list {
            .picture-list-item {
                display: inline-block;
                margin-right: 10px;
                cursor: pointer;
                border: 1px solid #ccc;
                width: 80px;
                height: 80px;
                overflow: hidden;
                img {
                    width: 100%;
                }
            }
        }
        .tip {
            color: red;
            position: relative;
            top: -2px;
        }
    }
</style>