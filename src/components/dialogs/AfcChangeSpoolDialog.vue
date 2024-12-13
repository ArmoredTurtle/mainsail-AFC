<template>
    <div>
        <v-dialog v-model="showDialog" width="800" persistent :fullscreen="isMobile">
            <panel
                :title="$t('Panels.SpoolmanPanel.ChangeSpool')"
                :icon="mdiAdjust"
                card-class="afc-change-spool-dialog"
                :margin-bottom="false">
                <template #buttons>
                    <v-btn icon tile @click="close">
                        <v-icon>{{ mdiCloseThick }}</v-icon>
                    </v-btn>
                </template>

                <v-card-title>
                    <v-text-field
                        v-if="spoolManagerUrl"
                        v-model="search"
                        :append-icon="mdiMagnify"
                        :label="$t('Panels.SpoolmanPanel.Search')"
                        outlined
                        dense
                        hide-details
                        style="max-width: 300px" />
                    <v-spacer />
                    <v-btn
                        :title="$t('Panels.SpoolmanPanel.EjectSpool')"
                        class="px-2 minwidth-0 ml-3"
                        :loading="loadings.includes('ejectSpool')"
                        @click="ejectSpool">
                        <v-icon>{{ mdiEject }}</v-icon>
                    </v-btn>
                    <v-btn
                        v-if="spoolManagerUrl"
                        :title="$t('Panels.SpoolmanPanel.Refresh')"
                        class="px-2 minwidth-0 ml-3"
                        :loading="loadings.includes('refreshSpools')"
                        @click="refreshSpools">
                        <v-icon>{{ mdiRefresh }}</v-icon>
                    </v-btn>
                    <v-btn
                        v-if="spoolManagerUrl"
                        :title="$t('Panels.SpoolmanPanel.OpenSpoolManager')"
                        class="px-2 minwidth-0 ml-3"
                        @click="openSpoolManager">
                        <v-icon>{{ mdiDatabase }}</v-icon>
                    </v-btn>
                </v-card-title>
                <v-card-text class="px-0 pb-0">
                    <v-data-table
                        v-if="spoolManagerUrl"
                        :headers="headers"
                        :items="spools"
                        item-key="id"
                        :search="search"
                        sort-by="last_used"
                        :sort-desc="true"
                        :custom-filter="customFilter">
                        <template #no-data>
                            <div class="text-center">{{ $t('Panels.SpoolmanPanel.NoSpools') }}</div>
                        </template>
                        <template #no-results>
                            <div class="text-center">{{ $t('Panels.SpoolmanPanel.NoResults') }}</div>
                        </template>

                        <template #item="{ item }">
                            <SpoolmanChangeSpoolDialogRow
                                :key="item.id"
                                :spool="item"
                                :max_id_digits="max_spool_id_digits"
                                @set-spool="setSpool" />
                        </template>
                    </v-data-table>

                    <div v-else>
                        <v-container>
                            <v-row align="center">
                                <!-- Color Picker with Label -->
                                <v-col cols="12" md="6" class="d-flex justify-center align-center">
                                    <div class="color-picker-container">
                                        <label class="color-picker-label"><h3>Filament Color</h3></label>
                                        <v-color-picker v-model="spoolColor" flat class="color-picker" />
                                    </div>
                                </v-col>

                                <!-- Right Aligned Text Boxes and Button -->
                                <v-col cols="12" md="6">
                                    <div class="text-box-container" style="display: flex; flex-direction: row">
                                        <!-- Text Boxes -->
                                        <v-row>
                                            <v-col cols="12">
                                                <v-text-field
                                                    v-model="spoolColor"
                                                    :label="$t('Panels.AfcSpoolmanPanel.SpoolColor')"
                                                    outlined />
                                            </v-col>
                                            <v-col cols="12">
                                                <v-text-field
                                                    v-model="filamentType"
                                                    :label="$t('Panels.AfcSpoolmanPanel.FilamentType')"
                                                    outlined />
                                            </v-col>
                                            <v-col cols="12">
                                                <v-text-field
                                                    v-model="remainingWeight"
                                                    :label="$t('Panels.AfcSpoolmanPanel.RemainingWeight')"
                                                    outlined
                                                    type="number" />
                                            </v-col>
                                        </v-row>
                                    </div>
                                </v-col>
                            </v-row>
                            <v-row style="width: 100%; margin-top: 16px">
                                <v-col cols="12">
                                    <v-btn color="primary" block @click="updateSpool">
                                        {{ $t('Panels.AfcSpoolmanPanel.UpdateSpool') }}
                                    </v-btn>
                                </v-col>
                            </v-row>
                        </v-container>
                    </div>
                </v-card-text>
            </panel>
        </v-dialog>
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins, Prop, Watch } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import Panel from '@/components/ui/Panel.vue'
import { mdiCloseThick, mdiAdjust, mdiDatabase, mdiMagnify, mdiRefresh, mdiEject } from '@mdi/js'
import { ServerSpoolmanStateSpool } from '@/store/server/spoolman/types'
import SpoolmanChangeSpoolDialogRow from '@/components/dialogs/SpoolmanChangeSpoolDialogRow.vue'

@Component({
    components: { SpoolmanChangeSpoolDialogRow, Panel },
})
export default class AfcChangeSpoolDialog extends Mixins(BaseMixin) {
    @Prop({ type: Object, required: true }) laneData!: any
    mdiAdjust = mdiAdjust
    mdiCloseThick = mdiCloseThick
    mdiDatabase = mdiDatabase
    mdiMagnify = mdiMagnify
    mdiRefresh = mdiRefresh
    mdiEject = mdiEject

    filamentType = ''
    remainingWeight = 0
    spoolColor = '#ffffff'

    @Prop({ required: true }) declare readonly showDialog: boolean

    search = ''

    get spools(): ServerSpoolmanStateSpool[] {
        return this.$store.state.server.spoolman.spools ?? []
    }

    get max_spool_id_digits(): number {
        const max_id = this.$store.state.server.spoolman.spools.reduce(
            (x: number, s: ServerSpoolmanStateSpool) => Math.max(x, s.id),
            0
        )

        return max_id.toString().length
    }

    get headers() {
        return [
            {
                text: ' ',
                align: 'start',
                sortable: false,
            },
            {
                text: this.$t('Panels.SpoolmanPanel.Filament'),
                align: 'start',
                value: 'filament.name',
                sortable: false,
            },
            {
                text: this.$t('Panels.SpoolmanPanel.Material'),
                align: 'center',
                value: 'filament.material',
            },
            {
                text: this.$t('Panels.SpoolmanPanel.LastUsed'),
                align: 'end',
                value: 'last_used',
            },
            {
                text: this.$t('Panels.SpoolmanPanel.Weight'),
                align: 'end',
                value: 'remaining_weight',
            },
        ]
    }

    get spoolManagerUrl() {
        return this.$store.state.server.config.config?.spoolman?.server ?? null
    }

    updateSpool() {
        console.log('Updating spool with the following data:')
        console.log('Filament Type:', this.filamentType)
        console.log('Remaining Weight:', this.remainingWeight)
        console.log('Spool Color:', this.spoolColor)

        if (this.laneData != null) {
            const cleanedColor = this.spoolColor.replace('#', '')

            const setColor = `SET_COLOR LANE=${this.laneData.laneName} COLOR=${cleanedColor}`
            const setWeight = `SET_WEIGHT LANE=${this.laneData.laneName}  WEIGHT=${this.remainingWeight}`
            const setMaterial = `SET_MATERIAL LANE=${this.laneData.laneName}  MATERIAL=${this.filamentType}`

            this.$nextTick(async () => {
                try {
                    await this.$store.dispatch('printer/sendGcode', setColor)
                    await this.$store.dispatch('printer/sendGcode', setWeight)
                    await this.$store.dispatch('printer/sendGcode', setMaterial)
                } catch (error) {
                    console.error('Failed to send G-code:', error)
                }
            })

            this.close()
        }
    }

    manualyClearSpool() {
        if (this.laneData != null) {
            const cleanedColor = this.spoolColor.replace('#', '')

            const setColor = `SET_COLOR LANE=${this.laneData.laneName} COLOR=000000`
            const setWeight = `SET_WEIGHT LANE=${this.laneData.laneName}  WEIGHT=0`
            const setMaterial = `SET_MATERIAL LANE=${this.laneData.laneName}  MATERIAL=`

            this.$nextTick(async () => {
                try {
                    await this.$store.dispatch('printer/sendGcode', setColor)
                    await this.$store.dispatch('printer/sendGcode', setWeight)
                    await this.$store.dispatch('printer/sendGcode', setMaterial)
                } catch (error) {
                    console.error('Failed to send G-code:', error)
                }
            })
        }
    }

    openSpoolManager() {
        window.open(this.spoolManagerUrl, '_blank')
    }

    mounted() {
        this.refresh()
    }

    refresh() {
        this.$store.dispatch('server/spoolman/refreshSpools')
    }

    close() {
        this.$emit('close')
    }

    ejectSpool() {
        if (this.laneData != null) {
            const setSpoolId = `SET_SPOOL_ID LANE=${this.laneData.laneName} SPOOL_ID=`
            const unloadLane = `LANE_UNLOAD LANE=${this.laneData.laneName}`
            console.log('Dispatching G-code:', setSpoolId)
            console.log('Dispatching G-code:', unloadLane)

            this.$nextTick(async () => {
                try {
                    await this.$store.dispatch('printer/sendGcode', setSpoolId)
                    console.log('SET_SPOOL_ID sent successfully')

                    await this.$store.dispatch('printer/sendGcode', unloadLane)
                    console.log('UNLOAD_LANE sent successfully')
                } catch (error) {
                    console.error('Failed to send G-code:', error)
                }
            })

            // Spoolman not active manualy clear the spool info
            if (!this.spoolManagerUrl) {
                this.manualyClearSpool()
            }

            this.$emit('fetch-spool')
            this.close()
        }
    }

    refreshSpools() {
        this.$store.dispatch('server/spoolman/refreshSpools')
    }

    customFilter(value: any, search: string, item: ServerSpoolmanStateSpool): boolean {
        if (search.trim().startsWith('web+spoolman:s-')) {
            const spoolId = parseInt(search.split('-')[1] ?? -1)
            return item.id === spoolId
        }

        const querySplits = search.toLowerCase().split(' ')
        const searchArray = [
            item.id.toString(),
            item.comment,
            item.filament.name,
            item.filament.vendor.name,
            item.filament.material,
            item.location,
        ]

        for (const query of querySplits) {
            const result = searchArray.some((q) => q?.toLowerCase().includes(query))

            if (!result) return false
        }

        return true
    }

    setSpool(spool: any) {
        console.log('Spool set as:', spool)
        console.log('Lane data received:', this.laneData)
        const gcode = `SET_SPOOL_ID LANE=${this.laneData.laneName} SPOOL_ID=${spool.id}`
        console.log('Dispatching G-code:', gcode)

        this.$nextTick(async () => {
            try {
                await this.$store.dispatch('printer/sendGcode', gcode)
                console.log('G-code sent successfully')
            } catch (error) {
                console.error('Failed to send G-code:', error)
            }
        })
        this.$emit('fetch-spool')
        this.close()
    }

    initializeFields() {
        if (this.laneData) {
            console.log('Initializing with laneData:', this.laneData)
            this.filamentType = this.laneData.material || ''
            this.remainingWeight = this.laneData.weight || 0
            this.spoolColor = this.laneData.color || '#ffffff'
        }
    }

    resetFields() {
        this.filamentType = ''
        this.remainingWeight = 0
        this.spoolColor = '#ffffff'
    }

    @Watch('showDialog')
    onShowDialogChanged(newVal: boolean) {
        if (newVal) {
            if (this.spoolManagerUrl) {
                this.search = ''
            }
        }
    }

    @Watch('laneData', { immediate: true })
    onLaneDataChanged() {
        if (this.showDialog && this.laneData && !this.spoolManagerUrl) {
            this.initializeFields()
        }
    }
}
</script>
<style scoped>
.color-picker-container {
    display: flex;
    flex-direction: column;
    margin: 10px;
}

.color-picker-label {
    font-weight: bold;
    margin-bottom: 8px;
}
</style>
