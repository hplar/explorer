<template>
    <span :class="{small: sm}">
        <span  class="text-monospace">{{amount | hexToVal(dec) | balance}}</span>
        <small v-if="sym && showSym" class="text-secondary">{{sym}}</small>
    </span>
</template>
<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator'
@Component
export default class Amount extends Vue {
    @Prop({ default: 0 })
    amount!: string | number

    @Prop({ default: '' })
    sym!: string

    @Prop({ default: true })
    showSym!: boolean
    @Prop()
    dec!: number

    @Prop({default:false})
    sm!: boolean
}
</script>
