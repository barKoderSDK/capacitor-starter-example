<template>
    <div id="container">
        <div id="barkoderView" ref="barkoderView"></div>

        <div class="btnContainer">
            <button class="actionButton" @click="startScanning" :disabled="isScanning">
                Start Scanning
            </button>
            <button class="actionButton" @click="stopScanning" :disabled="!isScanning">
                Stop Scanning
            </button>
            <button v-if="isScanning" class="actionButton" @click="toggleFlash">
                Toggle Flash
            </button>
        </div>

        <div v-if="scannedResult" class="resultContainer">
            <p>Result:
                <a :href="scannedResult.textualData"> {{ scannedResult.textualData }} </a>
            </p>
            <p>Type: {{ scannedResult.type }}</p>
            <img v-if="scannedResult?.thumbnailImage" class="resultImage" :src="scannedResult?.thumbnailImage" alt="Scanned Thumbnail" />
        </div>
    </div>
</template>

<script>
import { ref } from 'vue';
import { Barkoder, BarcodeType } from 'barkoder-capacitor';

export default {
    setup() {
        const barkoderViewRef = ref('');
        const scannedResult = ref(null);
        const flashEnabled = ref(false);
        const isScanning = ref(false);

        Barkoder.addListener('barkoderResultEvent', (barkoderResult) => {
            scannedResult.value = {
                textualData: barkoderResult.textualData,
                type: barkoderResult.barcodeTypeName,
                thumbnailImage: "data:image/jpeg;base64," + barkoderResult.resultThumbnailAsBase64,
            }
            isScanning.value = false;
            Barkoder.stopScanning()
        });

        const setActiveBarcodeTypes = () => {
            Barkoder.setBarcodeTypeEnabled({ type: BarcodeType.code128, enabled: true });
            Barkoder.setBarcodeTypeEnabled({ type: BarcodeType.ean13, enabled: true });
        };

        const setBarkoderSettings = () => {
            Barkoder.setRegionOfInterestVisible({ value: true });
            Barkoder.setRegionOfInterest({ left: 5, top: 5, width: 90, height: 90 });
            Barkoder.setCloseSessionOnResultEnabled({ enabled: false });
            Barkoder.setImageResultEnabled({ enabled: true });
            Barkoder.setBarcodeThumbnailOnResultEnabled({ enabled: true });
            Barkoder.setBeepOnSuccessEnabled({ enabled: true });
            Barkoder.setPinchToZoomEnabled({ enabled: true });
            Barkoder.setZoomFactor({ value: 2.0 });
        };

        const startScanning = async () => {
            scannedResult.value = {
                textualData: null,
                type: null,
                thumbnailImage: null,
            }
            isScanning.value = true;

            const boundingRect = await barkoderViewRef.value.getBoundingClientRect();
            await Barkoder.registerWithLicenseKey({ licenseKey: 'YOUR_LICENSE_KEY' });
            await Barkoder.initialize({
                width: Math.round(boundingRect.width),
                height: Math.round(boundingRect.height),
                x: Math.round(boundingRect.x),
                y: Math.round(boundingRect.y),
            });
            setBarkoderSettings();
            setActiveBarcodeTypes();
            await Barkoder.startScanning();
        };

        const stopScanning = async () => {
            scannedResult.value = {
                textualData: null,
                type: null,
                thumbnailImage: null,
            }
            isScanning.value = false;
            await Barkoder.stopScanning();
        };

        const toggleFlash = () => {
            flashEnabled.value = !flashEnabled.value;
            Barkoder.setFlashEnabled({ enabled: flashEnabled.value });
        };

        return {
            barkoderViewRef,
            startScanning,
            stopScanning,
            scannedResult,
            toggleFlash,
            isScanning
        }
    },
    mounted: function () {
        this.barkoderViewRef = this.$refs.barkoderView
    }
}

</script>

<style scoped>
#container {
    margin-top: 50px;
    text-align: center;
    position: absolute;
    padding: 10px;
    left: 0;
    right: 0;
    top: 0;
}

#container p {
    font-size: 16px;
    line-height: 22px;
    color: #8c8c8c;
    margin: 0;
}

#container a {
    text-decoration: none;
    color: #007bff;
    word-wrap: break-word;
}

#barkoderView {
    height: 380px;
}

.btnContainer {
    display: flex;
    flex-direction: column;
    gap: 10px;
    justify-content: center;
    margin-top: 20px;
}

.actionButton {
    width: 100%;
    height: 40px;
    font-size: 16px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}
.actionButton:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}

.resultContainer {
    margin-top: 20px;
    width: 100%;
}

.resultImage {
    width: 100%;
    max-width: 200px;
    height: auto;
    margin-top: 10px;
    border-radius: 5px;
}
</style>