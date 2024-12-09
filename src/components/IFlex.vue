<template>
    <div
        idm-ctrl="drag_container"
        class="idm-layout-iflex"
        :id="moduleObject.id"
        :idm-ctrl-id="moduleObject.id"
    >
        <div
            class="drag_container"
            v-for="item in cells"
            :key="item.id"
            @click="gridClickHandle(item.id)"
            idm-ctrl-inner
            :idm-ctrl-id="moduleObject.id"
            :idm-container-index="item.id"
            :idm-refresh-container="`flex-${item.id}`"
        >
            <!--统一的插槽写法，主要用于vue组件，其他语言的脚手架可忽略-->
            <slot :name="moduleObject.id + item.id"></slot>
        </div>
    </div>
</template>
<script>
export default {
    name: 'IFlex',
    data() {
        return {
            moduleObject: this._moduleObject || {},
            propData: this._propData?.compositeAttr ||
                this.$root?.propData?.compositeAttr || {
                    direction: 'row',
                    cell: 2,
                },
            innerAttr:
                this._propData?.innerAttr ||
                this.$root?.propData?.innerAttr ||
                [],
        }
    },
    props: {
        _moduleObject: Object,
        _propData: Object,
    },
    computed: {
        cells() {
            return Array.from({ length: this.propData.cell }).map(
                (_, index) => ({
                    id: index + 1,
                    name: `flex-${index + 1}`,
                })
            )
        },
    },
    created() {
        this.moduleObject = this._moduleObject || this.$root.moduleObject
        this.convertAttrToStyleObject()
    },
    mounted() {
        this._moduleObject &&
            IDM.callBackComponentMountComplete?.apply(this, [
                this._moduleObject,
            ])
    },
    methods: {
        /**
         * 提供父级组件调用的刷新prop数据组件
         */
        propDataWatchHandle(propData) {
            this.propData = propData.compositeAttr || {}
            this.innerAttr = propData.innerAttr || []
            this.convertAttrToStyleObject()
        },
        gridClickHandle(index) {
            const colAttrObject = this.innerAttr.filter(
                item => item.containerIndex == index
            )
            if (colAttrObject && colAttrObject.length > 0) {
                if (colAttrObject[0].dataAttr.clickFunction?.length) {
                    IDM.invokeCustomFunctions.apply(this, [
                        colAttrObject[0].dataAttr.clickFunction,
                        {
                            index,
                        },
                    ])
                }
            }
        },
        /**
         * 把属性转换成样式对象
         */
        convertAttrToStyleObject() {
            var styleObject = {}
            const keyList = [
                'width',
                'height',
                'box',
                'border',
                'font',
                'layout',
                'boxShadow',
                'bgColor',
                'gap',
                'justifyContent',
                'alignItems',
                'direction',
            ]
            for (const iKey in keyList) {
                const key = keyList[iKey]
                if (this.propData.hasOwnProperty.call(this.propData, key)) {
                    const element = this.propData[key]
                    if (!element && element !== false && element != 0) {
                        continue
                    }
                    switch (key) {
                        case 'width':
                        case 'height':
                            styleObject[key] = element
                            break
                        case 'box':
                            IDM.style.setBoxStyle(styleObject, element)
                            break
                        case 'border':
                            IDM.style.setBorderStyle(styleObject, element)
                            break
                        case 'font':
                            IDM.style.setFontStyle(styleObject, element)
                            break
                        case 'layout':
                            IDM.style.setLayoutStyle(styleObject, element)
                            break
                        case 'boxShadow':
                            styleObject['box-shadow'] = element
                            break
                        case 'bgColor':
                            if (element && element.hex8) {
                                styleObject['background-color'] =
                                    IDM.hex8ToRgbaString(element.hex8) +
                                    '  !important'
                            }
                            break
                        case 'gap':
                            styleObject[
                                'gap'
                            ] = `${element.inputVal}${element.selectVal}`
                            break
                        case 'justifyContent':
                            styleObject['justify-content'] = element
                            break
                        case 'alignItems':
                            styleObject['align-items'] = element
                            break
                        case 'direction':
                            styleObject['flex-direction'] = element
                            break
                    }
                }
            }
            if (!this.propData.bgList?.bgList?.length) {
                IDM.style.setBackgroundStyle(styleObject, this.propData)
            } else if (Object.keys(this.propData.bgList.style).length) {
                Object.assign(styleObject, this.propData.bgList.style)
            }
            let styleList = []
            styleList.push({
                selector: this.moduleObject.id,
                style: styleObject,
            })
            if (this.innerAttr && this.innerAttr.length > 0) {
                this.innerAttr.forEach(element => {
                    this.convertInnerAttrToStyleObject(
                        element.dataAttr,
                        element.containerIndex,
                        styleList
                    )
                })
            }
            IDM.setStyleObjectToPageHead(
                this.moduleObject.id + '_codestyle',
                styleList
            )
        },
        /**
         * 把属性参数转换成内部容器样式对象
         */
        convertInnerAttrToStyleObject(propData, index, styleList) {
            var styleObject = {}
            const colAttrObject = this.innerAttr.filter(
                item => item.containerIndex == index
            )
            const keyList = [
                'width',
                'height',
                'box',
                'border',
                'font',
                'layout',
                'boxShadow',
                'bgColor',
                'flex',
            ]
            for (const iKey in keyList) {
                const key = keyList[iKey]
                if (propData.hasOwnProperty.call(propData, key)) {
                    const element = propData[key]
                    if (!element && element !== false && element != 0) {
                        continue
                    }
                    switch (key) {
                        case 'width':
                            styleObject['width'] = element
                            if (
                                this.propData.widthRatioFixed &&
                                this.propData.direction == 'row' &&
                                colAttrObject.flex > 0
                            ) {
                                styleObject['width'] = 0
                            }
                            break
                        case 'height':
                            styleObject['height'] = element
                            if (
                                this.propData.widthRatioFixed &&
                                this.propData.direction == 'column' &&
                                colAttrObject.flex > 0
                            ) {
                                styleObject['height'] = 0
                            }
                            break
                        case 'box':
                            IDM.style.setBoxStyle(styleObject, element)
                            break
                        case 'border':
                            IDM.style.setBorderStyle(styleObject, element)
                            break
                        case 'font':
                            IDM.style.setFontStyle(styleObject, element)
                            break
                        case 'layout':
                            IDM.style.setLayoutStyle(styleObject, element)
                            break
                        case 'boxShadow':
                            styleObject['box-shadow'] = element
                            break
                        case 'bgColor':
                            if (element && element.hex8) {
                                styleObject['background-color'] =
                                    IDM.hex8ToRgbaString(element.hex8) +
                                    '  !important'
                            }
                            break
                        case 'flex':
                            styleObject['flex'] = element
                            break
                    }
                }
            }
            if (!propData.bgList?.bgList?.length) {
                IDM.style.setBackgroundStyle(styleObject, propData)
            } else if (Object.keys(propData.bgList.style).length) {
                Object.assign(styleObject, propData.bgList.style)
            }
            styleList.push({
                selector:
                    this.moduleObject.id +
                    ` .drag_container[idm-ctrl-id="${this.moduleObject.id}"][idm-container-index="${index}"]`,
                style: styleObject,
            })
        },
    },
}
</script>
<style lang="scss" scoped>
.idm-layout-iflex {
    display: flex;
}
</style>
