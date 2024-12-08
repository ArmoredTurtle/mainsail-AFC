<template>
    <div class="afc-panel-wrapper">
        <panel :icon="mdiAdjust"
               :title="'Automated Filament Control'"
               card-class="afc-panel"
               :collapsible="true"
               :expanded="true">
            <template #buttons>
                <v-btn icon tile :title="'Refresh AFC Spools'" @click="fetchSpoolData">
                    <v-icon>{{ mdiRefresh }}</v-icon>
                </v-btn>
                <v-menu :offset-y="true" :close-on-content-click="true" left>
                    <template #activator="{ on, attrs }">
                        <v-btn icon tile v-bind="attrs" v-on="on">
                            <v-icon>{{ mdiDotsVertical }}</v-icon>
                        </v-btn>
                    </template>
                    <v-list>
                        <v-list-item>
                            <v-radio-group v-model="selectedStyle">
                                <v-radio label="Style 1" :value="1" @change="saveStyleIndex(1)"></v-radio>
                            </v-radio-group>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>
            <div class="status-wrapper">
                <div class="tool-status">
                    <strong>Tool:</strong>
                    <span :class="{
                            'status-light': true,
                            'status-green': toolStartSensorStatus,
                            'status-red': !toolStartSensorStatus,
                        }"></span>
                </div>
                <div class="buffer-status">
                    <span v-if="systemData?.extruders?.extruder?.buffer_status">
                        <strong>Buffer:</strong> {{ bufferStatus() }}
                    </span>
                </div>
            </div>
            <div v-for="(unit, unitName) in unitsData"
                 :key="unitName"
                 class="unit-section">
                <v-expansion-panels>
                    <v-expansion-panel>
                        <v-expansion-panel-header>
                            <div class="unit-header"
                                 style="display: flex; align-items: center; gap: 10px">
                                <BoxTurtleIcon v-if="unitsData[unitName]?.system?.type === 'Box_Turtle'"
                                               style="width: 12%; height: 12%; margin-left: 10px;" />
                                <h2 class="unit-title" style="margin: 0">
                                    {{ String(unitName).replace(/_/g, " ") }} |
                                </h2>
                                <span class="hub-status">
                                    <span><strong>Hub</strong></span>
                                    <span :class="{
                                            'status-light': true,
                                            'status-green': getHubStatus({unitName : unitName}),
                                            'status-red': !getHubStatus({unitName : unitName}),
                                            }">
                                    </span>
                                </span>
                            </div>
                        </v-expansion-panel-header>
                        <v-expansion-panel-content style="padding-top: 0em;">
                            <div class="spool-container">
                                <div v-for="(spool, index) in unit.spools"
                                     style="margin-top: 0px;"
                                     :key="index"
                                     class="spool-card">
                                    <div class="filament-reel"
                                         style="padding: 1rem"
                                         :key="index"
                                         @click="openChangeSpoolDialog(spool)">
                                        <FilamentReelIcon v-if="spool.load"
                                                          :color="spool.color"
                                                          style="width: 60%; float: left"
                                                          class="mr-3" />
                                        <FilamentReelIcon v-else
                                                          style="width: 60%; float: left"
                                                          :color='spool.empty'
                                                          class="mr-3" />

                                    </div>
                                    <div class="spool-header">
                                        <span :class="{
                                                'status-light': true,
                                                'status-not-ready': determineStatus(spool) === 'Not Ready',
                                                'status-prep': determineStatus(spool) === 'Prep',
                                                'status-ready': determineStatus(spool) === 'Ready',
                                                'status-in-tool': determineStatus(spool) === 'In Tool',
                                            }">
                                        </span>
                                        <h3>{{ spool.laneName }}</h3>
                                        <div class="spacer"></div>
                                        <select :name="'map-' +spool.laneName"
                                                class="afclist"
                                                @change="handleMapChange($event, spool)">
                                            <template v-for="option in mapList" v-bind:value="option">
                                                <template v-if="option === spool.map">
                                                    <option :value="option" selected>
                                                        {{ option }}
                                                    </option>
                                                </template>
                                                <template v-else>
                                                    <option :value="option">
                                                        {{ option }}
                                                    </option>
                                                </template>
                                            </template>
                                        </select>
                                    </div>
                                    <infinity-icon v-if="spool.runout_lane === 'NONE'"
                                                   :color="infispoolNo""
                                                   height="20px"
                                                   width="20px"
                                                   version="1.1"
                                                   id="Capa_1"
                                                   style="float: right; margin-top: 2px; margin-left: 5px;"
                                                   viewBox="0 0 60 60" xml:space="preserve"/>
                                    <infinity-icon v-else
                                                   height="20px"
                                                   :color="infispoolYes"
                                                   width="20px"
                                                   version="1.1"
                                                   id="Capa_1"
                                                   style="float: right; margin-top: 2px; margin-left: 5px;"
                                                   viewBox="0 0 60 60" xml:space="preserve"/>
                                    
                                    <select :name="'run-' +spool.laneName"
                                            class="afclist"
                                            @change="handleRunoutChange($event, spool)">
                                        <template v-for="option in laneList" v-bind:value="option">
                                            <template v-if="option === spool.runout_lane">
                                                <option :value="option" selected>
                                                    {{ option }}
                                                </option>
                                            </template>
                                            <template v-else>
                                                <option :value="option">
                                                    {{ option }}
                                                </option>
                                            </template>
                                        </template>
                                    </select>
                                    <p v-if="spool.material">{{ spool.material }}</p>
                                    <p v-if="spool.weight">{{ spoolWeight(spool) }}</p>
                                </div>
                            </div>
                        </v-expansion-panel-content>
                    </v-expansion-panel>
                </v-expansion-panels>
            </div>

        </panel>
        <afc-change-spool-dialog :show-dialog="showChangeSpoolDialog"
                                 :index="index"
                                 :lane-data="selectedLane"
                                 @close="showChangeSpoolDialog = false"
                                 @fetch-spool="fetchSpoolData" />
    </div>
</template>

<script lang="ts">import { Component, Mixins } from "vue-property-decorator"
import BaseMixin from "@/components/mixins/base"
import Panel from "@/components/ui/Panel.vue"
import { mdiAdjust, mdiRefresh, mdiDotsVertical } from "@mdi/js"
import AfcChangeSpoolDialog from "@/components/dialogs/AfcChangeSpoolDialog.vue"
import FilamentReelIcon from '@/components/ui/FilamentReelIcon.vue'
import SpoolIcon from "@/components/ui/SpoolIcon.vue"
import { AFCRoot } from '@/store/server/afc'
import BoxTurtleIcon from "@/components/ui/BoxTurtleIcon.vue"
import InfinityIcon from '@/components/ui/InfinityIcon.vue'

@Component({
    components: {
        Panel,
        AfcChangeSpoolDialog,
        SpoolIcon,
        FilamentReelIcon,
        BoxTurtleIcon,
        InfinityIcon
    },
})
export default class AfcPanel extends Mixins(BaseMixin) {
    mdiAdjust = mdiAdjust;
    mdiRefresh = mdiRefresh;
    mdiDotsVertical = mdiDotsVertical;

    showChangeSpoolDialog = false;
    selectedLane: any = null; // This will hold data of the clicked lane

    spoolData: any[] = [];
    intervalId: number | null = null;
    systemData: any = null;
    index: number = 0;
    unitsData: any = {};

    selectedStyle: number = 1;
    styleIndex: number = 1;

    infispoolYes: any='#00FF00';
    infispoolNo: any='#FF0000';

    async mounted() {
        const savedStyleIndex = localStorage.getItem('styleIndex');
        if (savedStyleIndex !== null) {
            this.styleIndex = parseInt(savedStyleIndex, 10);
        }
        this.fetchSpoolData();
        this.intervalId = setInterval(this.fetchSpoolData, 5);
    }

    beforeDestroy() {
        if (this.intervalId) {
            clearInterval(this.intervalId);
        }
    }

    get afc_Data(): AFCRoot | null {
        return this.$store.state.printer.AFC
    }

    fetchSpoolData() {
        const afcData = this.afc_Data ? JSON.parse(JSON.stringify(this.afc_Data)) : null;

        if (afcData) {
            this.spoolData = this.extractLaneData(afcData);
            this.unitsData = this.groupByUnit(this.spoolData);
            this.systemData = afcData.system || {};
            for (const unitName in afcData) {
                if (afcData.hasOwnProperty(unitName) && unitName !== "system") {
                    if (this.unitsData[unitName]) {
                        this.unitsData[unitName].system = afcData[unitName].system || {};
                    }
                }
            }
        } else {
            this.spoolData = [];
            this.unitsData = {};
            this.systemData = {};
        }
    }

    extractLaneData(spools: any) {
        const lanes = [];
        const mapList = [];
        const laneList = [];
        this.mapList = [];
        this.laneList = [];
        if (spools && typeof spools === "object") {
            for (const unitName in spools) {
                if (spools.hasOwnProperty(unitName) && unitName !== "system") {
                    const unit = spools[unitName];
                    for (const laneKey in unit) {
                        if (
                            unit.hasOwnProperty(laneKey) &&
                            typeof unit[laneKey] === "object" &&
                            laneKey !== "system"
                        ) {
                            const laneData = unit[laneKey];
                            laneData.unitName = unitName;
                            laneData.laneName = laneKey;
                            laneData.empty = '#2e2e2e'
                            lanes.push(laneData);
                            laneList.push(laneKey);
                            mapList.push(unit[laneKey]['map']);
                        }
                    }
                }
            }
        }
        this.laneList=laneList.sort();
        this.laneList.unshift('NONE');
        this.mapList = mapList.sort();
        return lanes;
    }


    saveStyleIndex(changer: number){
        this.styleIndex = changer;
        localStorage.setItem('styleIndex', changer.toString());
    }

    private groupByUnit(spoolData: any[]) {
        const units: any = {};
        spoolData.forEach((spool) => {
            const unitName = spool.unitName;
            if (!units[unitName]) {
                units[unitName] = { spools: [] };
            }
            units[unitName].spools.push(spool);
        });
        return units;
    }

    openChangeSpoolDialog(spool: any) {
        this.selectedLane = { spool, laneName: spool.laneName };
        this.showChangeSpoolDialog = true;
    }

    bufferStatus() {
        return this.systemData?.extruders?.extruder?.buffer_status || false;
    }

    getHubStatus({ unitName }: { unitName: any }) {
        if (this.unitsData[unitName]?.system?.hub_loaded !== undefined) {
            return this.unitsData[unitName].system.hub_loaded;
        }
        return this.systemData?.hub_loaded || false;
    }

    get toolStartSensorStatus() {
        return this.systemData?.extruders?.extruder?.tool_start_sensor || false;
    }

    spoolWeight(spool: any) {
        const weight = parseInt(spool.weight, 10);
        return weight ? `${weight} g` : "";
    }

    handleRunoutChange(event: Event, spool: any) {
        const selectedValue = (event.target as HTMLSelectElement).value;
        console.log(`Selected value for ${spool.laneName}: ${selectedValue}`);

    //Example G-Code Call for you
        const gcode = `SET_RUNOUT LANE=${spool.laneName} RUNOUT=${selectedValue}`
        console.log('Dispatching G-code:', gcode)

        this.$nextTick(async () => {
            try {
                await this.$store.dispatch('printer/sendGcode', gcode)
                console.log('G-code sent successfully')
            } catch (error) {
                console.error('Failed to send G-code:', error)
            }
        })
    }

    handleMapChange(event: Event, spool: any) {
        const selectedValue = (event.target as HTMLSelectElement).value;
        console.log(`Selected value for ${spool.laneName}: ${selectedValue}`);

        //Example G-Code Call for you
        const gcode = `SET_MAP LANE=${spool.laneName} MAP=${selectedValue}`
        console.log('Dispatching G-code:', gcode)

        this.$nextTick(async () => {
            try {
                await this.$store.dispatch('printer/sendGcode', gcode)
                console.log('G-code sent successfully')
            } catch (error) {
                console.error('Failed to send G-code:', error)
            }
        })
    }
    private determineStatus(spool: any) {
        if (spool.prep) {
            if (spool.load) {
                if (this.systemData && this.systemData.current_load === spool.laneName) {
                    if (spool.spool_id == this.$store.state.server.spoolman.active_spool?.id) {
                        spool.weight = this.$store.state.server.spoolman.active_spool?.remaining_weight;
                    }
                    return "In Tool";
                }
                return "Ready";
            }
            return "Prep";
        }
        return "Not Ready";
    }
}</script>

<style scoped>
    .afc-panel-wrapper {
        margin-bottom: 24px;
    }

    .spool-container {
        display: flex;
        float: left;
        flex-wrap: wrap;
        justify-content: space-evenly;
        gap: 8px;
        padding: 8px;
        padding-top: 0px;
        margin-top: 0px;
    }

    .unit-title {
        font-size: 1.5em;
        margin-bottom: 16px;
        text-align: left;
    }

    .spool-card {
        background-color: #2e2e2e;
        border-radius: 8px;
        padding: 8px;
        padding-top: 0px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        flex: 1 1 calc(23% - 16px);
        max-width: 120px;
        min-width: 80px;
        min-height: 117px;
        position: relative;
        cursor: hand;
        transition: box-shadow 0.3s;
        margin-bottom: 0px;
        text-align: right;
    }

    .filament-reel {
        position: absolute;
        bottom: -10px;
        left: -10px;
    }

    .spool-card p {
        margin: 4px 0;
    }

    .spool-header {
        display: flex;
        justify-content: space-evenly;
        gap: 5px;
    }

    .afclist {
        background-color: #2e2e2e;
        color: white;
        text-align: right;
    }

    .status-wrapper {
        display: flex;
        float: left;
        flex-wrap: wrap;
        justify-content: space-evenly;
        border-bottom: 1px solid #ccc;
        padding-bottom: 10px;
        margin-bottom: 15px;
    }

    .status-light {
        width: 10px;
        height: 10px;
        border-radius: 50%;
        display: inline-block;
        margin-right: 5px;
        margin-left: 5px;
    }

    .status-green {
        background-color: rgb(24, 177, 24);
    }

    .status-red {
        background-color: red;
    }

    .hub-status {
        text-align: left;
        margin: 15px 0;
    }

    .buffer-status {
        display: block;
        margin-top: 5px;
    }

    .tool-status {
        text-align: center;
        margin-left: 15px;
    }

    .status-not-ready {
        background-color: red;
    }

    .status-prep {
        background-color: rgb(255, 255, 0);
    }

    .status-ready {
        background-color: rgb(26, 230, 26);
    }

    .status-in-tool {
        background-color: rgb(6, 197, 245);
    }
</style>
