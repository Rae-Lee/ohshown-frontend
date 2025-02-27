<template>
  <div class="create-factory-steps">
    <!-- AppBar for Mobile -->
    <v-app-bar fixed color="white" class="d-block d-md-none">
      <div
        class="btn-container"
        :class="{ hide: appState.createStepIndex === 1 }"
      >
        <v-btn icon @click="onBack" class="secondary--text">
          <v-icon>mdi-keyboard-backspace</v-icon>
        </v-btn>
      </div>

      <v-spacer></v-spacer>
      <v-toolbar-title class="secondary--text"
        >新增黑熊出沒痕跡 步驟 ({{ appState.createStepIndex }}/4)</v-toolbar-title
      >
      <v-spacer></v-spacer>

      <div class="btn-container">
        <v-dialog v-model="discardDialog" max-width="290">
          <template v-slot:activator="{ on, attrs }">
            <v-btn icon v-on="on" v-bind="attrs" class="secondary--text">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </template>
          <v-card>
            <v-card-title class="secondary--text"
              >放棄新增可疑黑熊出沒痕跡嗎？</v-card-title
            >
            <v-card-text
              >放棄新增可疑黑熊出沒痕跡時，你將遺失所有已輸入的資料。下次需重新填寫。</v-card-text
            >
            <v-container class="text-center">
              <v-btn
                width="100%"
                x-large
                rounded
                color="primary"
                @click="discardDialog = false"
                >繼續填寫資料</v-btn
              >
              <a
                class="d-block mt-4 text-decoration-underline"
                @click="cancelCreateFactory"
                >放棄新增</a
              >
            </v-container>
          </v-card>
        </v-dialog>
      </div>
    </v-app-bar>

    <!-- AppBar for Desktop -->
    <v-app-bar fixed color="white" v-show="$vuetify.breakpoint.mdAndUp">
      <v-toolbar-title>新增可疑黑熊出沒痕跡</v-toolbar-title>

      <div class="ml-15 desktop-step-item" @click="switchStep(1)">
        <span>選擇黑熊出沒痕跡位置</span>
        <v-icon class="mr-1">mdi-chevron-right</v-icon>
      </div>

      <div
        class="desktop-step-item"
        :class="{ inactive: appState.createStepIndex < 2 }"
        @click="switchStep(2)"
      >
        <span>上傳黑熊出沒痕跡照片</span>
        <v-icon class="mr-1">mdi-chevron-right</v-icon>
      </div>

      <div
        class="desktop-step-item"
        :class="{ inactive: appState.createStepIndex < 3 }"
        @click="switchStep(3)"
      >
        <span>填寫問券</span>
        <v-icon class="mr-1">mdi-chevron-right</v-icon>
      </div>

      <div
        class="desktop-step-item"
        :class="{ inactive: appState.createStepIndex < 4 }"
        @click="switchStep(4)"
      >
        <span>填寫聯絡資訊</span>
      </div>

      <v-spacer></v-spacer>

      <v-dialog
        v-model="discardDialog"
        max-width="290"
        v-if="$vuetify.breakpoint.mdAndUp"
      >
        <template v-slot:activator="{ on, attrs }">
          <v-btn v-on="on" v-bind="attrs" outlined> 取消新增 </v-btn>
        </template>
        <v-card>
          <v-card-title class="secondary--text"
            >放棄新增可疑黑熊出沒痕跡嗎？</v-card-title
          >
          <v-card-text
            >放棄新增可疑黑熊出沒痕跡時，你將遺失所有已輸入的資料。下次需重新填寫。</v-card-text
          >
          <v-container class="text-center">
            <v-btn
              width="100%"
              x-large
              rounded
              color="primary"
              @click="discardDialog = false"
              >繼續填寫資料</v-btn
            >
            <a
              class="d-block mt-4 text-decoration-underline"
              @click="cancelCreateFactory"
              >放棄新增</a
            >
          </v-container>
        </v-card>
      </v-dialog>
    </v-app-bar>

    <div class="create-factory-step-1" v-if="appState.createStepIndex === 1">
      <div
        class="select-location-container container-fluid pa-3 py-md-5 px-md-10"
        v-if="showLongLat"
        :class="{ desktop: $vuetify.breakpoint.mdAndUp }"
      >
        <div
          class="d-flex flex-md-row justify-space-between align-end align-md-center"
        >
          <div
            class="d-flex flex-column flex-md-row align-start align-md-center justify-space-between justify-md-start flex-grow-1"
          >
            <p class="mb-4 my-md-3 mr-md-10 wgs">以下經緯度版本為WGS84</p>

            <p
              class="font-weight-medium h5 mb-0 mb-1-md mr-md-10 location-text"
              v-if="!inlineLocationForm"
            >
              經度：{{ appState.mapLngLat[0].toFixed(7) }}
            </p>

            <p
              class="font-weight-medium h5 mb-0 mr-md-8 location-text"
              v-if="!inlineLocationForm"
            >
              緯度：{{ appState.mapLngLat[1].toFixed(7) }}
            </p>

            <v-btn
              rounded
              color="white"
              class="mr-2 d-none d-md-block primary--text"
              v-if="!inlineLocationForm"
              @click="toggleInlineLocationForm"
            >
              搜尋經緯度
            </v-btn>

            <div
              class="d-none d-md-flex justify-space-between align-center flex-grow-1"
              v-if="inlineLocationForm"
            >
              <div class="d-flex align-center">
                <p class="font-weight-medium h5 mb-0 mr-7">搜尋經緯度</p>

                <div
                  class="d-block mr-7"
                  style="height: 36px; width: 1px; background-color: #eaf3bf"
                />

                <h5 class="mr-2">經度</h5>

                <v-text-field
                  hide-details
                  class="mr-7"
                  outilned
                  solo
                  v-model="locationInputState.longitude"
                  placeholder="例：121.5231872"
                  dense
                />

                <h5 class="mr-2">緯度</h5>

                <v-text-field
                  hide-details
                  class="mr-7"
                  outilned
                  solo
                  v-model="locationInputState.latitude"
                  placeholder="例：25.0458344"
                  dense
                />

                <h5
                  class="text-decoration-underline mr-5"
                  style="cursor: pointer"
                  @click="clearLocationInput"
                >
                  清空
                </h5>

                <v-btn
                  rounded
                  color="white"
                  class="primary--text"
                  @click="setLocation"
                  >定位</v-btn
                >
              </div>

              <div class="d-flex align-center" v-if="inlineLocationForm">
                <div
                  class="d-block d-none d-md-block mx-7"
                  style="height: 36px; width: 1px; background-color: #eaf3bf"
                />

                <v-btn
                  icon
                  @click="toggleInlineLocationForm"
                  class="primary--text"
                >
                  <v-icon color="white">mdi-close</v-icon>
                </v-btn>
              </div>
            </div>
          </div>

          <v-dialog v-model="chooseLocationDialog" max-width="290">
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                rounded
                color="white"
                class="mr-2 d-md-none primary--text"
                v-on="on"
                v-bind="attrs"
              >
                搜尋經緯度
              </v-btn>
            </template>
            <v-card outlined class="primary">
              <v-card-text class="pt-4 white--text">
                <p>以下經緯度版本為WGS84</p>

                <h3 class="mb-2">經度</h3>

                <v-text-field
                  outilned
                  solo
                  v-model="locationInputState.longitude"
                  placeholder="例：121.5231872"
                />

                <h3 class="mb-2">緯度</h3>

                <v-text-field
                  outilned
                  solo
                  v-model="locationInputState.latitude"
                  placeholder="例：25.0458344"
                />

                <div class="d-flex justify-space-between mt-3 align-end">
                  <a
                    class="text-decoration-underline white--text"
                    style="height: 36px; line-height: 36px"
                    @click="clearLocationInput"
                    >清空</a
                  >
                  <v-btn
                    rounded
                    color="white"
                    class="primary--text"
                    @click="setLocation"
                    >定位</v-btn
                  >
                </div>
              </v-card-text>
            </v-card>
          </v-dialog>
        </div>
      </div>

      <div class="px-5 py-4">
        <v-btn
          rounded
          color="white"
          class="mr-2 primary--text"
          @click="toggleShowLongLat"
        >
          {{ showLongLat ? "隱藏經緯度" : "顯示經緯度" }}
        </v-btn>

        <switch-map-mode-button />

        <v-container
          fluid
          class="choose-location-btn-container d-flex justify-center"
          bottom="50"
        >
          <v-btn
            x-large
            rounded
            @click="chooseLocation"
            class="px-md-15"
            color="secondary"
          >
            選擇此地點
          </v-btn>
        </v-container>
      </div>
    </div>

    <image-upload-form
      v-if="appState.createStepIndex === 2"
      v-model="selectedImages"
      :uploading="imageUploadState.uploading"
      :error="imageUploadState.error"
      :previewImages="uploadedImages"
      :onClickRemoveImage="onClickRemoveImage"
      :valid="imageUploadFormValid"
      :submit="pageTransition.gotoNextCreate"
      :formState="formState"
    />

    <detail-form
      v-if="appState.createStepIndex === 3"
      :formState="formState"
      :previewImages="uploadedImages"
      :submit="pageTransition.gotoNextCreate"
    />

    <confirm-factory
      v-if="appState.createStepIndex === 4"
      :formState="formState"
      :previewImages="uploadedImages"
      :submit="submitFactory"
    />
  </div>
</template>

<script lang="ts">
import { createComponent, inject, ref, reactive } from '@vue/composition-api'

import { useAppState } from '../lib/appState'
import { useAlertState } from '../lib/useAlert'

import { MainMapControllerSymbol } from '../symbols'
import { MapFactoryController } from '../lib/map'
import { createFactory } from '../api'
import { useImageUpload } from '../lib/imageUpload'

import ImageUploadForm from './ImageUploadForm.vue'
import ConfirmFactory from './ConfirmFactory.vue'
import SwitchMapModeButton from './SwitchMapModeButton.vue'
import { FactoryPostData } from '../types'
import { useGA } from '../lib/useGA'
import { useBackPressed } from '../lib/useBackPressed'
import { useModalState } from '../lib/hooks'
import DetailForm from './DetailForm.vue'

export default createComponent({
  name: 'CreateFactorySteps',
  components: {
    ImageUploadForm,
    ConfirmFactory,
    SwitchMapModeButton,
    DetailForm
  },
  setup () {
    const [appState, { pageTransition, setFactoryLocation }] = useAppState()
    const [, { openCreateFactorySuccessModal }] = useModalState()
    const [, alertActions] = useAlertState()
    const { event } = useGA()

    const discardDialog = ref(false)

    const mapController = inject(
      MainMapControllerSymbol,
      ref<MapFactoryController>()
    )

    function cancelCreateFactory () {
      if (mapController.value) {
        mapController.value.mapInstance.setLUILayerVisible(false)
        discardDialog.value = false
        // TODO: clear form value
        pageTransition.closeFactoryPage()
      }
    }

    const onBack = () => {
      if (appState.createStepIndex === 1) {
        cancelCreateFactory()
      } else if (appState.createStepIndex === 2) {
        if (mapController.value) {
          pageTransition.previousCreateStep()
        }
      } else {
        pageTransition.previousCreateStep()
      }
    }
    useBackPressed(onBack)

    const createFactoryFormState = reactive({
      submitting: false,
      nickname: '',
      contact: '',
      others: '',
      name: '',

      // 表單基礎資訊
      type: undefined, // 通報類型
      alreadyOhshown: undefined,
      alreadyOhshownNumber: undefined,
      /** locale date */
      date: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
      /** locale time */
      time: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substring(11, 16),

      // 環境地貌
      groundTypes: [], // 土地類型
      vegetations: [], // 植被
      bearAttractors: [], // 附近可能吸引熊接近的物品

      // 個體資訊
      bearNumber: undefined, // 目擊個體數
      bearType: [], // 個體種類（成熊、幼熊）
      bearSize: [], // 體型
      bearSex: [], // 性別
      bearFeature: [], // 其他特徵

      // 黑熊行為與反應
      ohshownFeeling: '', // 看到熊當下，目擊者的感覺
      humanNumber: undefined, // 目擊熊當下人數
      humanBehavior: undefined, // 目擊者目擊當下在做什麼
      humanBehaviorText: '', // 目擊者目擊當下在做什麼-文字補充
      distance: undefined, // 目擊當下，目擊者與熊之間的距離
      bearBehavior: undefined, // 目擊當下，熊在做什麼
      bearBehaviorText: '', // 目擊當下，熊在做什麼-文字補充
      food: [], // 黑熊食物
      foodText: {}, // 黑熊食物-文字補充
      bearNotice: undefined, // 黑熊何時注意到人員存在
      bearNoticeMinutes: undefined, // 目擊後約X分鐘黑熊發現人員存在-數字補充
      humanReaction: [], // 目擊黑熊後，目擊者反應
      humanReactionText: '', // 目擊者反應-文字補充
      bearReaction: [], // 黑熊發現目擊者後，黑熊的反應
      bearReactionText: '', // 黑熊反應-文字補充
      humanHurt: undefined, // 是否有人受傷或意外發生
      humanHurtDescription: '', // 是否有人受傷或意外發生-文字補充

      // 發現黑熊痕跡
      traceType: undefined, // 痕跡類型
      traceTypeText: '', // 痕跡類型-文字補充
      freshness: undefined, // 新舊估計
      freshnessNumber: undefined, // 新舊估計-數字補充
      imageAvailable: undefined, // 是否提供影像檔案
      otherInfo: '', // 其他補充說明

      // 下一次，如果有機會的話
      ohShownAgain: undefined, // 目擊者是否希望以後再看到野外的黑熊
      ohShownAgainReason: '', // 目擊者是否希望以後再看到野外的黑熊-原因
      preventOhshownMethods: [], // 您知道以下哪些做法有助於減少遇到熊的機會，或避免不愉快地與熊相遇
      preventOhshownMethodsText: '', // 您知道以下哪些做法有助於減少遇到熊的機會，或避免不愉快地與熊相遇-文字補充
      surveyIfBearExist: undefined // 您是否會先了解您預計前往的地點有無黑熊出沒
    })

    const {
      selectedImages,
      imageUploadState,
      uploadedImages,
      onClickRemoveImage,
      imageUploadFormValid
    } = useImageUpload()

    const submitFactory = async () => {
      createFactoryFormState.submitting = true

      try {
        const [lng, lat]: number[] = appState.factoryLocation
        const bears = [...Array(createFactoryFormState.bearNumber || 0).keys()].map(index => ({
          bearType: createFactoryFormState.bearType[index],
          bearSize: createFactoryFormState.bearSize[index],
          bearSex: createFactoryFormState.bearSex[index],
          bearFeature: createFactoryFormState.bearFeature[index]
        }))
        const factory: FactoryPostData = {
          ...createFactoryFormState,
          bears,
          lng,
          lat,
          images: uploadedImages.value.map((i) => i.token),
          /** 遭遇時間戳記 */
          datetime: Date.parse(`${createFactoryFormState.date}T${createFactoryFormState.time}`)
        }

        event('createFactory', { lng, lat })
        const resultFactory = await createFactory(factory)
        if (mapController.value) {
          mapController.value.addFactories([resultFactory])
        }
        pageTransition.closeFactoryPage()
        openCreateFactorySuccessModal()
      } catch (e) {
        alertActions.showAlert('伺服器錯誤，請稍後再試')
      } finally {
        createFactoryFormState.submitting = false
      }
    }

    const showLongLat = ref(false)
    const toggleShowLongLat = () => {
      showLongLat.value = !showLongLat.value
    }

    const chooseLocationDialog = ref(false)
    const inlineLocationForm = ref(false)
    const toggleInlineLocationForm = () => {
      inlineLocationForm.value = !inlineLocationForm.value
    }
    const locationInputState = reactive({
      longitude: '',
      latitude: ''
    })
    const clearLocationInput = () => {
      locationInputState.longitude = ''
      locationInputState.latitude = ''
    }
    const setLocation = () => {
      if (!mapController.value) return

      const lng = parseFloat(locationInputState.longitude)
      const lat = parseFloat(locationInputState.latitude)
      if (
        !locationInputState.longitude ||
        !locationInputState.latitude ||
        isNaN(lng) ||
        isNaN(lat)
      ) {
        // TODO: show invalid input error
        chooseLocationDialog.value = false
      } else {
        let zoom = mapController.value.mapInstance.map.getView().getZoom()
        if (!zoom || zoom <= 17) {
          zoom = 17
        }
        mapController.value.mapInstance.setCoordinate(lng, lat, zoom)
        chooseLocationDialog.value = false
      }
    }

    const switchStep = (num: number) => {
      const backSteps = appState.createStepIndex - num

      for (let i = 0; i < backSteps; i++) {
        onBack()
      }
    }

    return {
      appState,
      pageTransition,
      formState: createFactoryFormState,

      cancelCreateFactory,
      onBack,
      chooseLocation () {
        if (!mapController.value) return

        if (!appState.canPlaceFactory) {
          alertActions.showAlert(
            '此地點不在農地範圍內，\n請回報在農地範圍內的黑熊出沒痕跡。'
          )
          return
        }

        mapController.value.mapInstance.setLUILayerVisible(false)

        setFactoryLocation(appState.mapLngLat as [number, number])

        pageTransition.nextCreateStep()
      },
      selectedImages,
      imageUploadState,
      uploadedImages,
      onClickRemoveImage,
      imageUploadFormValid,
      discardDialog,
      submitFactory,
      showLongLat,
      toggleShowLongLat,
      chooseLocationDialog,
      inlineLocationForm,
      locationInputState,
      clearLocationInput,
      toggleInlineLocationForm,
      setLocation,
      switchStep
    }
  }
})
</script>

<style lang="scss" scoped>
@import "@/styles/variables";

.btn-container.hide {
  visibility: hidden;
  pointer-events: none;
}

.create-factory-steps {
  width: 100%;
  height: 100%;

  .create-factory-step-1 {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;

    .choose-location-btn-container {
      left: 0;
      position: fixed;
      bottom: 50px;
    }
  }

  .select-location-container {
    background-color: $light-green-color;
    color: white;

    .wgs {
      font-size: 12px;
    }

    .location-text {
      font-size: 14px;
    }

    &.desktop {
      .wgs {
        font-size: 14px;
      }

      .location-text {
        font-size: 16px;
      }
    }
  }
}

.desktop-step-item {
  &:not(.inactive) {
    cursor: pointer;
  }

  user-select: none;
  color: $dark-green-color;

  &.inactive {
    opacity: 0.5;
  }

  .v-icon {
    margin-top: -3px;
  }
}
</style>
