<template>
    <div class="extruder-item">
        <div class="extruder-container">
            <div class="left-section">
                <v-tooltip top>
                    <template #activator="{ on, attrs }">
                        <span
                            v-bind="attrs"
                            :class="{
                                'status-light': true,
                                success: tool.tool_start_sensor,
                                error: !tool.tool_start_sensor,
                            }"
                            v-on="on"></span>
                    </template>
                    <span>{{ preSensorStatus }}</span>
                </v-tooltip>
                <span class="tool-name">{{ toolName }}</span>
                <v-tooltip top>
                    <template #activator="{ on, attrs }">
                        <span
                            v-if="tool.tool_end_sensor !== null"
                            v-bind="attrs"
                            :class="{
                                'status-light': true,
                                success: tool.tool_end_sensor,
                                error: !tool.tool_end_sensor,
                            }"
                            v-on="on"></span>
                    </template>
                    <span>{{ postSensorStatus }}</span>
                </v-tooltip>
            </div>
            <div class="buffer-info">{{ tool.buffer }}: {{ tool.buffer_status }}</div>
            <div class="lane-status">
                {{ $t('Panels.AfcPanel.LaneLoaded') }}:
                <span :class="tool.lane_loaded !== '' ? 'primary--text' : 'error--text'">
                    {{ tool.lane_loaded !== '' ? tool.lane_loaded : $t('Panels.AfcPanel.LaneLoadedNone') }}
                </span>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins, Prop } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import { Extruder } from '@/store/server/afc/types'

@Component({})
export default class AfcExtruderToolsItem extends Mixins(BaseMixin) {
    @Prop({ type: Object, required: true }) readonly tool!: Extruder
    @Prop({ type: String, required: true }) readonly toolName!: string

    get preSensorStatus(): string {
        const status = this.tool.tool_start_sensor
            ? this.$t('Panels.AfcPanel.Detected')
            : this.$t('Panels.AfcPanel.Empty')
        return `${this.$t('Panels.AfcPanel.PreExtruderSensor')} - ${status}`
    }

    get postSensorStatus(): string {
        const status = this.tool.tool_end_sensor
            ? this.$t('Panels.AfcPanel.Detected')
            : this.$t('Panels.AfcPanel.Empty')
        return `${this.$t('Panels.AfcPanel.PostExtruderSensor')} - ${status}`
    }
}
</script>

<style scoped>
.extruder-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    flex-wrap: wrap;
}

.left-section {
    display: flex;
    align-items: center;
    gap: 8px;
    min-width: 120px;
    flex-shrink: 0;
}

.tool-name {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 150px;
}

.buffer-info {
    text-align: center;
    flex: 1 0;
    min-width: 120px;
}

.lane-status {
    text-align: right;
    min-width: 120px;
    flex: 0;
}

.status-light {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    display: inline-block;
    margin: 0 5px;
}
</style>
