<template>
    <div class="image-cropper-stage">
        <loading-screen v-if="loading"></loading-screen>
        <img v-once :ref="(el) => cropperStage = el" v-show="!loading" />
        <form method="POST" v-show="false">
            <input type="hidden" name="height" v-bind:value="imageCropper.data.height">
            <input type="hidden" name="width" v-bind:value="imageCropper.data.width">
            <input type="hidden" name="x" v-bind:value="imageCropper.data.x">
            <input type="hidden" name="y" v-bind:value="imageCropper.data.y">
        </form>
    </div>
</template>

<script setup lang="ts">
import {inject, onMounted, ref, getCurrentInstance} from "vue";
import ImageCropperApp from "../image-cropper/ImageCropperApp";
import $ from "jquery";
import Cropper from 'cropperjs';
import 'cropperjs/dist/cropper.min.css';

const imageCropper = inject<ImageCropperApp>('imageCropper');

let cropper: Cropper = null;
let loading = false;
let initialDataSetEvent = false;
let cropperStage = null;

onMounted(() => {
    loading = true;
    let element = cropperStage;
    element.src = imageCropper.data.url;
    cropper = new Cropper(element, {
        ready: () => {
            initialDataSetEvent = true;
            if (imageCropper.data.ratio) {
                cropper.setAspectRatio(imageCropper.data.ratio);
            }
            if (imageCropper.data.getData()) {
                cropper.setData(imageCropper.data.getData());
            }
            loading = false;

            const instance = getCurrentInstance();
            instance?.proxy?.$forceUpdate();
        }
    });

    element.addEventListener('crop', () => {
        let cropperData = cropper.getData();

        let changed = (value1: number, value2: number) => {
            return Math.abs(value1 - value2) > 0.00001;
        };

        if (changed(imageCropper.data.height, cropperData.height)) {
            imageCropper.data.changed = true;
            imageCropper.data.height = cropperData.height;
        }

        if (changed(imageCropper.data.width, cropperData.width)) {
            imageCropper.data.changed = true;
            imageCropper.data.width = cropperData.width;
        }

        if (changed(imageCropper.data.x, cropperData.x)) {
            imageCropper.data.changed = true;
            imageCropper.data.x = cropperData.x;
        }

        if (changed(imageCropper.data.y, cropperData.y)) {
            imageCropper.data.changed = true;
            imageCropper.data.y = cropperData.y;
        }

        if (initialDataSetEvent) {
            initialDataSetEvent = false;
            imageCropper.data.changed = false;
        }
    });

    $(document).on('image-cropper-zoom-in', () => {
        cropper.zoom(0.1);
    });

    $(document).on('image-cropper-zoom-out', () => {
        cropper.zoom(-0.1);
    });

    $(document).on('image-cropper-reset', () => {
        cropper.reset();
    });
})

</script>

<style lang="css" scoped>
    .image-cropper-stage { height: 100%;position:relative; }
</style>
