<template>
    <div class="parkinggenerator">
        <div class="pg__header">
            <button @click="zoomIn">+</button>
            <button @click="zoomOut">-</button>
            <button @click="loadSiteConfig">Load Site Config File</button>
            <input type="file" ref="configFileInput" accept="application/json" @change="changeConfigFile" hidden />
            <button @click="openSiteInfoModal">Set Site Information</button>
            <button @click="loadImage">Load Image</button>
            <input type="file" ref="imgFileInput" accept="image/*" @change="changeImageFile" hidden />
            <button @click="drawSlot">Draw Sloat Box</button>
            <button @click="openRemoveSlotModal">Remove Slot Box</button>
            <button @click="openCameraInfoModal">Set Camera</button>
            <button @click="openRemoveCameraModal">Remove Camera</button>
            <button @click="saveSiteConfig">Save Site Config File</button>
        </div>
        <div class="pg__content">
            <div
                id="pgcanvas"
                :class="['pg__canvas', { 'pg__canvas--draw': canDraw || canPlaceCamera }]"
                v-dragscroll
            >
            </div>
            <div class="pg__data">
                <div class="pg__sitename">
                    <label>Site Name:</label>
                    <input type="text" v-model="siteName" readonly />
                </div>
                <div class="pg__tables">
                    <div class="pg__slots">
                        <table>
                            <tr>
                                <th>NO</th>
                                <th>Slot Name</th>
                                <th>Slot Code</th> 
                                <th>Slot Owner</th>
                            </tr>
                            <tr
                                v-for="(slot, index) in slots"
                                :key="slot.slotId"
                                :class="[{ 'row-selected': slot === selectedSlot }]"
                                @click="selectSlotTableRow(slot)"
                            >
                                <td>{{ index+1 }}</td>
                                <td>{{ slot.slotName }}</td>
                                <td>{{ slot.slotId }}</td>
                                <td>{{ slot.slotOwner }}</td>
                            </tr>
                        </table>
                    </div>
                    <div class="pg__cameras">
                        <table>
                            <tr>
                                <th>No</th>
                                <th>Camera Name</th>
                                <th>Camera Link</th> 
                            </tr>
                            <tr
                                v-for="(camera, index) in cameras"
                                :key="camera.cameraName"
                                :class="[{ 'row-selected': camera === selectedCamera }]"
                                @click="selectCameraTableRow(camera)"
                            >
                                <td>{{ index+1 }}</td>
                                <td>{{ camera.cameraName }}</td>
                                <td>{{ camera.cameraLink }}</td>
                            </tr>
                        </table>
                    </div>
                </div>
            </div>
        </div>
        <site-info-modal
          v-if="siteInfoModalOpen"
          :data="{ siteName }"
          @confirm="setSiteInfo"
          @close="siteInfoModalOpen = false"
        />
        <slot-info-modal
          v-if="slotInfoModalOpen"
          :data="{ slotName: '', slotId: '', slotOwner: '' }"
          @confirm="setSlotInfo"
          @close="cancelAddSlot"
        />
        <remove-slot-modal
          v-if="removeSlotModalOpen"
          :data="selectedSlot"
          @confirm="removeSlot"
          @close="removeSlotModalOpen = false"
        />
        <camera-info-modal
          v-if="cameraInfoModalOpen"
          :data="{ cameraName: '', cameraLink: '' }"
          @confirm="setCameraInfo"
          @close="cancelAddCamera"
        />
        <remove-camera-modal
          v-if="removeCameraModalOpen"
          :data="selectedCamera"
          @confirm="removeCamera"
          @close="removeCameraModalOpen = false"
        />
    </div>
</template>

<script>
import * as d3 from 'd3';
import SiteInfoModal from './SiteInfoModal.vue'
import SlotInfoModal from './SlotInfoModal.vue'
import RemoveSlotModal from './RemoveSlotModal.vue'
import CameraInfoModal from './CameraInfoModal.vue'
import RemoveCameraModal from './RemoveCameraModal.vue'
import { dragscroll } from 'vue-dragscroll'
import objectLink from '../../../ui/mixins/object-link';

export default {
    inject: ['openmct', 'domainObject'],
    directives: {
        dragscroll
    },
    components: {
        SiteInfoModal,
        SlotInfoModal,
        RemoveSlotModal,
        CameraInfoModal,
        RemoveCameraModal
    },
    data: function () {
        return {
            currentDomainObject: this.domainObject,
            siteInfoModalOpen: false,
            slotInfoModalOpen: false,
            removeSlotModalOpen: false,
            cameraInfoModalOpen: false,
            removeCameraModalOpen: false,
            canvasWidth: 0,
            canvasHeight: 0,
            imageWidth: 0,
            imageHeight: 0,
            svg: null,
            gSlot: null,
            gCamera: null,
            canDraw: false,
            canPlaceCamera: false,
            zoomLevel: 1,
            siteName: '',
            slots: [],
            cameras: [],
            selectedSlot: {},
            selectedCamera: {}
        };
    },
    methods:{
        zoomIn() {
            if (this.svg && this.zoomLevel < 3) {
                this.zoomLevel += 0.1;
                this.svg.attr('transform', `scale(${this.zoomLevel})`);
                var x = parseFloat(this.svg.style('left'));
                var y = parseFloat(this.svg.style('top'));
                var w = parseFloat(this.svg.attr('width'));
                var h = parseFloat(this.svg.attr('height'));
                x += w * 0.05;
                y += h * 0.05;
                this.svg.style('left', x);
                this.svg.style('top', y);    
            }
        },
        zoomOut() {
            if (this.svg && this.zoomLevel > 1) {
                this.zoomLevel -= 0.1;
                this.svg.attr('transform', `scale(${this.zoomLevel})`);
                var x = parseFloat(this.svg.style('left'));
                var y = parseFloat(this.svg.style('top'));
                var w = parseFloat(this.svg.attr('width'));
                var h = parseFloat(this.svg.attr('height'));
                x -= w * 0.05;
                y -= h * 0.05;
                this.svg.style('left', x);
                this.svg.style('top', y);                
            }
        },
        loadSiteConfig() {
            this.$refs.configFileInput.click();
        },
        changeConfigFile(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (event) => {
                    const siteConfig = JSON.parse(event.target.result);
                    this.clearAll();
                    this.prepareCanvas(siteConfig['image file name'], (dragger) => {
                        const ratio = Math.min(this.canvasWidth / this.imageWidth, this.canvasHeight / this.imageHeight);
                        this.siteName = siteConfig['site name'];
                        this.slots = siteConfig.slots.map(s => ({
                            slotName: s['slot name'],
                            slotId: s['slot id'],
                            slotOwner: s['slot owner']
                        }));
                        this.cameras = siteConfig.cameras.map(c => ({
                            cameraName: c['camera name'],
                            cameraLink: c['camera link']
                        }));
                        siteConfig.slots.forEach((s) => {
                            const points = s['slot location'].map(n => n * ratio);
                            const pointsStr = points.map(n => String(n)).join(',');
                            const gSlot = this.svg.append('g').attr('data-slotid', s['slot id']);
                            gSlot.append('polygon')
                                .attr('points', pointsStr)
                                .style('fill', 'none')
                                .style('stroke', '#fe0000');
                            for (let i = 0; i < points.length; i+=2) {
                                gSlot.append('circle')
                                    .attr('cx', points[i])
                                    .attr('cy', points[i+1])
                                    .attr('r', 4)
                                    .attr('fill', '#fe0000')
                                    .attr('stroke', '#fe0000')
                                    .attr('is-handle', 'true')
                                    .attr('class', 'slot-point')
                                    .style('cursor', 'move')
                                    .call(dragger);
                            }
                        });
                        siteConfig.cameras.forEach((c) => {
                            const [x, y] = c['camera location'];
                            const gCamera = this.svg.append("svg:image")
                                .attr('xlink:href', require('./camera_button.png'))
                                .attr('width', 62.5)
                                .attr('height', 83)
                                .attr('x', x * ratio)
                                .attr('y', y * ratio)
                                .attr('data-cameraname', c['camera name'])
                                .attr('transform', 'translate(-26, -83)');
                            gCamera.on('click', () => this.openCamera(gCamera));
                        });
                    });
                };
                reader.readAsText(file);
            }
        },
        openSiteInfoModal() {
            this.siteInfoModalOpen = true;
        },
        setSiteInfo(data) {
            this.siteInfoModalOpen = false;
            this.siteName = data.siteName;
        },
        clearAll() {
            if (this.svg) {
                this.svg.remove();
            }
            this.svg = null;
            this.gSlot = null;
            this.gCamera = null;
            this.canDraw = false;
            this.canPlaceCamera = false;
            this.zoomLevel = 1;
            this.siteName = '';
            this.slots = [];
            this.cameras = [];
            this.selectedSlot = {};
            this.selectedCamera = {};
        },
        prepareCanvas(imgUrl, callback) {
            const img = new Image();
            img.onload = () => {
                const { width, height } = img;
                this.imageWidth = width;
                this.imageHeight = height;

                const elCanvas = document.querySelector('#pgcanvas');
                this.canvasWidth = elCanvas.clientWidth;
                this.canvasHeight = elCanvas.clientHeight;

                const ratio = Math.min(this.canvasWidth / this.imageWidth, this.canvasHeight / this.imageHeight);
                const svgWidth = this.imageWidth * ratio;
                const svgHeight = this.imageHeight * ratio;

                this.svg = d3.select('#pgcanvas').append('svg')
                    .attr('width', svgWidth)
                    .attr('height', svgHeight)
                    .attr('style', `position: absolute; left: ${(this.canvasWidth - svgWidth) * 0.5}px; top: ${(this.canvasHeight - svgHeight) * 0.5}px`);
                this.svg.append("svg:image")
                    .attr('xlink:href', imgUrl)
                    .attr('preserveAspectRatio', 'none')
                    .attr('width', svgWidth)
                    .attr('height', svgHeight)
                    .attr('x', 0)
                    .attr('y', 0);

                const self = this;
                let g = null, drawing = false, dragging = false, points = [], startPoint = [];

                function handleDrag() {
                    if (drawing || self.canDraw || self.canPlaceCamera) return;
                    let dragCircle = d3.select(this), newPoints = [];
                    dragCircle
                        .attr('cx', d3.event.x)
                        .attr('cy', d3.event.y);
                    const poly = d3.select(this.parentNode).select('polygon');
                    const circles = d3.select(this.parentNode).selectAll('circle');
                    circles.each(function() {
                        const circle = d3.select(this);
                        newPoints.push([circle.attr('cx'), circle.attr('cy')]);
                    });
                    poly.attr('points', newPoints);

                    dragging = true;
                }

                function closePolygon() {
                    self.svg.select('g.drawPoly').remove();
                    self.gSlot = self.svg.append('g');
                    self.gSlot.append('polygon')
                        .attr('points', points)
                        .style('fill', 'none')
                        .style('stroke', '#fe0000');
                    for (let i = 0; i < points.length; i++) {
                        self.gSlot.selectAll('circles')
                          .data([points[i]])
                          .enter()
                          .append('circle')
                          .attr('cx', points[i][0])
                          .attr('cy', points[i][1])
                          .attr('r', 4)
                          .attr('fill', '#fe0000')
                          .attr('stroke', '#fe0000')
                          .attr('is-handle', 'true')
                          .attr('class', 'slot-point')
                          .style('cursor', 'move')
                          .call(dragger);
                    }
                    points.splice(0);
                    drawing = false;
                    self.canDraw = false;
                    self.slotInfoModalOpen = true;
                }

                const dragger = d3.drag()
                    .on('drag', handleDrag)
                    .on('end', () => {
                        dragging = false;
                    });

                this.svg.on('click', function() {
                    if (!self.canPlaceCamera) return;
                    const [x, y] = d3.mouse(this);
                    self.gCamera = self.svg.append("svg:image")
                        .attr('xlink:href', require('./camera_button.png'))
                        .attr('width', 62.5)
                        .attr('height', 83)
                        .attr('x', x)
                        .attr('y', y)
                        .attr('transform', 'translate(-26, -83)');
                    self.gCamera.on('click', () => self.openCamera(self.gCamera));
                    self.canPlaceCamera = false;
                    self.cameraInfoModalOpen = true;
                });
                this.svg.on('mouseup', function() {
                    if (dragging) return;
                    if (!self.canDraw) return;
                    drawing = true;
                    startPoint = [d3.mouse(this)[0], d3.mouse(this)[1]];
                    if (self.svg.select('g.drawPoly').empty()) g = self.svg.append('g').attr('class', 'drawPoly');
                    if (d3.event.target.hasAttribute('is-handle')) {
                        closePolygon();
                        return;
                    };
                    points.push(d3.mouse(this));
                    g.select('polyline').remove();
                    g.append('polyline').attr('points', points)
                        .style('fill', 'none')
                        .attr('stroke', '#fe0000');
                    for (let i = 0; i < points.length; i++) {
                        g.append('circle')
                            .attr('cx', points[i][0])
                            .attr('cy', points[i][1])
                            .attr('r', 4)
                            .attr('fill', 'yellow')
                            .attr('stroke', '#fe0000')
                            .attr('is-handle', 'true')
                            .style('cursor', 'pointer');
                    }
                });
                this.svg.on('mousemove', function() {
                    if (!drawing) return;
                    if (!self.canDraw) return;
                    const g = d3.select('g.drawPoly');
                    g.select('line').remove();
                    g.append('line')
                        .attr('x1', startPoint[0])
                        .attr('y1', startPoint[1])
                        .attr('x2', d3.mouse(this)[0] + 2)
                        .attr('y2', d3.mouse(this)[1])
                        .attr('stroke', '#fe0000')
                        .attr('stroke-width', 3);
                });

                if (callback) {
                    callback.call(this, dragger);
                }
            };
            img.src = imgUrl;
        },
        loadImage() {
            this.$refs.imgFileInput.click();
        },
        changeImageFile(event) {
            const file = event.target.files[0];
            if (file) {
                this.clearAll();

                const reader = new FileReader();
                reader.onload = (e) => {
                    this.prepareCanvas(e.target.result);
                }
                reader.readAsDataURL(file);
            }
        },
        drawSlot() {
            if (this.svg) {
                this.canDraw = true;
            }
        },
        setSlotInfo(data) {
            this.slots.push(data);
            this.gSlot.attr('data-slotid', data.slotId);
            this.slotInfoModalOpen = false;
        },
        cancelAddSlot() {
            this.gSlot.remove();
            this.gSlot = null;
            this.slotInfoModalOpen = false;
        },
        openRemoveSlotModal() {
            if (this.slots.includes(this.selectedSlot)) {
                this.removeSlotModalOpen = true;
            }
        },
        selectSlotTableRow(slot) {
            this.selectedSlot = slot;
        },
        removeSlot() {
            const slotId = this.selectedSlot.slotId;
            this.slots = this.slots.filter(s => s !== this.selectedSlot);
            this.selectedSlot = {};
            this.svg.selectAll('g')
                .filter(function() {
                    return d3.select(this).attr('data-slotid') === slotId;
                })
                .remove();
            this.removeSlotModalOpen = false;
        },
        openCameraInfoModal() {
            if (this.svg) {
                this.canPlaceCamera = true;
            }
        },
        setCameraInfo(data) {
            this.cameras.push(data);
            this.gCamera.attr('data-cameraname', data.cameraName);
            this.cameraInfoModalOpen = false;
        },
        cancelAddCamera() {
            this.gCamera.remove();
            this.gCamera = null;
            this.cameraInfoModalOpen = false;
        },
        openRemoveCameraModal() {
            if (this.cameras.includes(this.selectedCamera)) {
                this.removeCameraModalOpen = true;
            }
        },
        selectCameraTableRow(camera) {
            this.selectedCamera = camera;
        },
        removeCamera() {
            const cameraName = this.selectedCamera.cameraName;
            this.cameras = this.cameras.filter(c => c !== this.selectedCamera);
            this.selectedCamera = {};
            this.svg.selectAll('image')
                .filter(function() {
                    return d3.select(this).attr('data-cameraname') === cameraName;
                })
                .remove();
            this.removeCameraModalOpen = false;
        },
        saveSiteConfig() {
            const ratio = Math.min(this.canvasWidth / this.imageWidth, this.canvasHeight / this.imageHeight);
            const configJson = {
                "file format version": "1.0",
                "site name": this.siteName,
                "image file name": this.svg.select('image').attr('href'),
                "slots": this.slots.map((slot) => ({
                    "slot name": slot.slotName,
                    "slot id": slot.slotId,
                    "slot owner": slot.slotOwner,
                    "slot location": this.svg.selectAll('polygon')
                        .filter(function() {
                            return d3.select(this.parentNode).attr('data-slotid') === slot.slotId;
                        }).attr('points').split(',').map(s => Math.floor((Number(s) / ratio) * 100) / 100)
                })),
                "cameras": this.cameras.map((camera) => {
                    const node = this.svg.selectAll('image')
                        .filter(function() {
                            return d3.select(this).attr('data-cameraname') === camera.cameraName;
                        });
                    return {
                        "camera name": camera.cameraName,
                        "camera link": camera.cameraLink,
                        "camera location": [Math.floor((Number(node.attr('x')) / ratio) * 100) / 100, Math.floor((Number(node.attr('y')) / ratio) * 100) / 100]
                    }
                })
            }
            this.downloadJSON(configJson);
        },
        downloadJSON(json) {
            const downloadAnchorNode = document.createElement('a');
            downloadAnchorNode.setAttribute('href', "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(json)));
            downloadAnchorNode.setAttribute('download', 'download.json');
            document.body.appendChild(downloadAnchorNode);
            downloadAnchorNode.click();
            downloadAnchorNode.remove();
        },
        openCamera(el) {
            const cameraName = el.attr('data-cameraname');
            const camera = this.cameras.filter(c => c.cameraName === cameraName)[0];
            if (camera) {
                console.log(camera.cameraLink);
            }
        }
    }
};
</script>
