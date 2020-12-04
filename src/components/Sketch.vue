<template>
    <div class="sketch" :class="{active: isActive}" ref="sketch" v-on:dblclick="onDoubleClick">
        <template v-for="sprite in this.sprites">
            <Moveable v-if="sprite.selected" :key="sprite.id" :id="sprite.id"
                class="sprite-wrapper"
                :style="stylePosition(sprite)"
                @drag="handleDrag"
                @dragEnd="handleDragEnd(sprite)"
                v-bind="moveable">
                <svg-box class="sprite" :ref="'sprite-' + sprite.id" :sourse="sprite.sourse"></svg-box>
            </Moveable>
            <div v-else class="sprite-wrapper placed" :style="stylePosition(sprite)" :key="sprite.id" 
                :id="sprite.id" >
                <svg-box 
                    :sourse="sprite.sourse" 
                    :ref="'sprite-' + sprite.id" 
                    class="sprite" 
                    v-on:select="onSelectItem(sprite)"
                ></svg-box>
            </div>
        </template>
        <template v-for="(textSprite, index) in this.texts">
            <div class="text-sprite" :style="getTextSpritePosition(textSprite)" :key="index">{{textSprite.text}}</div>
        </template>
        <div class="text-input-wrapper" v-if="text.isVisible" :style="textInputPosition">
            <input type="text" v-on:blur="setText" v-model="text.value" />
        </div>
    </div>
</template>

<script>
import SvgBox from './SvgBox.vue';
import Moveable from 'vue-moveable';
export default {
    name: 'Sketch',
    components: { Moveable, SvgBox },
    props: {
        sprites: {
            type: Array,
            default: () => []
        },
        texts: {
            type: Array,
            default: () => []
        },
        isActive: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            moveable: {
                draggable: true,
                throttleDrag: 0,
                resizable: false,
                throttleResize: 1,
                keepRatio: true,
                scalable: false,
                throttleScale: 0,
                rotatable: false,
                throttleRotate: 0,
            },
            drag: {
                left: null,
                top: null
            },
            text: {
                isVisible: false,
                position: {
                    left: 0,
                    right: 0
                },
                value: ''
            }
        }
    },
    methods: {
        handleDrag({ target, left, top }) {
            target.style.left = `${left}px`;
            target.style.top = `${top}px`;
            this.drag.left = left;
            this.drag.top = top;
            //console.log('handleDrag', target, left, top);
        },
        handleDragEnd(payload) {
            this.$emit('change-position', {
                sprite: payload,
                top: this.drag.top,
                left: this.drag.left
            });
        },
        stylePosition(sprite) {
            return 'left: ' + sprite.left + 'px; top: ' + sprite.top + 'px;';
        },
        onSelectItem(payload) {
            this.$emit('select', payload);
        },
        onDoubleClick(event) {
            if (this.text.isVisible) {
                return;
            }
            this.text.value = '';
            this.text.isVisible = true;
            this.text.position.left = event.clientX - 14;
            this.text.position.top = event.offsetY - 10;
        },
        setText(event) {
            this.text.isVisible = false;
            if (!this.text.value) {
                return;
            }
            this.$emit('put-text', {
                text: this.text.value,
                top: this.text.position.top + 4,
                left: this.text.position.left + 4
            });
        },
        getTextSpritePosition(textSprite) {
            return 'left: ' + textSprite.left + 'px; top: ' + textSprite.top + 'px;';
        }
    },
    computed: {
        textInputPosition() {
            return 'left: ' + this.text.position.left + 'px; top: ' + this.text.position.top + 'px;';
        }
    }
}
</script>

<style lang="scss">
.sketch {
    position: absolute;
    top: 0;
    width: 100%;
    bottom: 0;
    box-sizing: border-box;
    -webkit-user-select: none;  /* Chrome all / Safari all */
    -moz-user-select: none;     /* Firefox all */
    -ms-user-select: none;      /* IE 10+ */
    user-select: none;  
    .sprite-wrapper {
        width: 110px;
        height: 110px;
        position: absolute;
    }
    &.active {
        background-color: #cccccc0f;
    }
    .text-input-wrapper {
        position: absolute;
        input {
            background-color: #33333366;
            border: 0 none;
            color: #ffb100;
            padding: 8px;
            border-bottom: 1px solid #ffb100;
            &:focus {
                border: 0 none;
                outline: none;
                border-bottom: 1px solid #ffb100;
            }
        }
    }
    .text-sprite {
        position: absolute;
        color: #ffb100;
        font-size: 14px;
    }
}
</style>