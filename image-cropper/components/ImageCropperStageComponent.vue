<template>
    <div class="image-cropper-stage">
        <loading-screen v-if="loading"></loading-screen>
        <img v-once ref="cropper-stage" v-show="!loading" />
        <form method="POST" v-show="false">
            <input type="hidden" name="height" v-bind:value="$imageCropper.data.height">
            <input type="hidden" name="width" v-bind:value="$imageCropper.data.width">
            <input type="hidden" name="x" v-bind:value="$imageCropper.data.x">
            <input type="hidden" name="y" v-bind:value="$imageCropper.data.y">
        </form>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from "vue-property-decorator";
import * as Cropper from 'cropperjs';
import 'cropperjs/dist/cropper.min.css';
import * as $ from 'jquery';

@Component()
export default class ImageCropperStageComponent extends Vue
{
    
    cropper: Cropper;
    loading = false;

    mounted()
    {
        this.loading = true;
        let element = <HTMLElement>this.$refs['cropper-stage'];
        element.src = this.$imageCropper.data.url;
        this.cropper = new Cropper(element, {
            ready: () => {
                if(this.$imageCropper.data.ratio) {
                    this.cropper.setAspectRatio(this.$imageCropper.data.ratio);
                }
                if(this.$imageCropper.data.getData()) {
                    this.cropper.setData(this.$imageCropper.data.getData());
                }
                this.loading = false;
                this.$forceUpdate();
            }
        });

        element.addEventListener('crop', () => {
            this.$imageCropper.data.height = this.cropper.getData().height;
            this.$imageCropper.data.width = this.cropper.getData().width;
            this.$imageCropper.data.x = this.cropper.getData().x;
            this.$imageCropper.data.y = this.cropper.getData().y;
            this.$imageCropper.data.changed = true;
        });

        $(document).on('image-cropper-zoom-in', () => {
            this.cropper.zoom(0.1);
        });

        $(document).on('image-cropper-zoom-out', () => {
            this.cropper.zoom(-0.1);
        });

        $(document).on('image-cropper-reset', () => {
            this.cropper.reset();
        });
    }
}
</script>

<style lang="css" scoped>
    .image-cropper-stage { height: 100%;position:relative; }
</style>


