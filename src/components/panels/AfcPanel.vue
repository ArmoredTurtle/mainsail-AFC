<template>
    <div class="afc-panel-wrapper">
        <panel
            :icon="mdiAdjust"
            :title="'Automated Filament Control'"
            card-class="afc-panel"
            :collapsible="true"
            :expanded="true"
        >
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
                                <v-radio label="Style 2" :value="0" @change="saveStyleIndex(0)"></v-radio>
                            </v-radio-group>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>
            <div class="status-wrapper">
                <div class="tool-status">
                    <strong>Tool:</strong>
                    <span
                        :class="{
                            'status-light': true,
                            'status-green': toolStartSensorStatus,
                            'status-red': !toolStartSensorStatus,
                        }"
                    ></span>
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
                                <div class="spool-container" style="margin-top: 0px">
                                <div v-for="(spool, index) in unit.spools"
                                    :key="index"
                                    class="spool-card">
                                    <div class="filament-reel"
                                        style="padding: 1rem"
                                        :key="index"
                                        @click="openChangeSpoolDialog(spool)">
                                        <svg viewBox="0 0 235 500" preserveAspectRatio="xMinYMin meet" width="23.5" height="50" xml:space="preserve" xmlns="http://www.w3.org/2000/svg">
                                            <path style="stroke:#6e0b30;stroke-width:0;stroke-dasharray:none;stroke-linecap:butt;stroke-dashoffset:0;stroke-linejoin:miter;stroke-miterlimit:4;fill:#c08f4f;fill-rule:nonzero;opacity:1" vector-effect="non-scaling-stroke" transform="matrix(.58757 0 0 3.94769 197.135 250.047)" d="M0-63.27c34.925 0 63.27 28.345 63.27 63.27 0 34.925-28.345 63.27-63.27 63.27-34.925 0-63.27-28.345-63.27-63.27 0-34.925 28.345-63.27 63.27-63.27z" />
                                            <path style="stroke:#6e0b30;stroke-width:0;stroke-dasharray:none;stroke-linecap:butt;stroke-dashoffset:0;stroke-linejoin:miter;stroke-miterlimit:4;fill:#c08f4f;fill-rule:nonzero;opacity:1" vector-effect="non-scaling-stroke" transform="matrix(.38158 0 0 3.46232 197.135 250.047)" d="M0-63.27c34.925 0 63.27 28.345 63.27 63.27 0 34.925-28.345 63.27-63.27 63.27-34.925 0-63.27-28.345-63.27-63.27 0-34.925 28.345-63.27 63.27-63.27z" />
                                            <path v-if="spool.load" class="filament-reel"
                                                    :style="{
                                                        fill: spool.color,
                                                        stroke: '#000',
                                                        strokeWidth: 0,
                                                        strokeDasharray: 'none',
                                                        strokeLinecap: 'butt',
                                                        strokeDashoffset: 0,
                                                        strokeLinejoin: 'miter',
                                                        strokeMiterlimit: 4,
                                                        fillRule: 'nonzero',
                                                        opacity: 1
                                                    }"
                                                    vector-effect="non-scaling-stroke"
                                                    transform="matrix(2.07364 0 0 3.3577 117.295 250.047)"
                                                    d="M-38.503-65.24h77.006V65.24h-77.006z" />
                                                <g transform="matrix(.58757 0 0 3.94769 37.454 250.047)">
                                                    <filter id="a" y="-.057" height="1.114" x="-.057" width="1.272">
                                                        <feGaussianBlur in="SourceAlpha" stdDeviation="3" />
                                                        <feOffset dx="20" result="oBlur" />
                                                        <feFlood flood-color="rgb(0,0,0)" flood-opacity=".67" />
                                                        <feComposite in2="oBlur" operator="in" />
                                                        <feMerge>
                                                            <feMergeNode />
                                                            <feMergeNode in="SourceGraphic" />
                                                        </feMerge>
                                                    </filter>
                                                    <path style="stroke:#6e0b30;stroke-width:0;stroke-dasharray:none;stroke-linecap:butt;stroke-dashoffset:0;stroke-linejoin:miter;stroke-miterlimit:4;fill:#c08f4f;fill-rule:nonzero;opacity:1;filter:url(#a)" vector-effect="non-scaling-stroke" d="M0-63.27c34.925 0 63.27 28.345 63.27 63.27 0 34.925-28.345 63.27-63.27 63.27-34.925 0-63.27-28.345-63.27-63.27 0-34.925 28.345-63.27 63.27-63.27z" />
                                                    <path style="stroke:#6e0b30;stroke-width:0;stroke-dasharray:none;stroke-linecap:butt;stroke-dashoffset:0;stroke-linejoin:miter;stroke-miterlimit:4;fill:#c08f4f;fill-rule:nonzero;opacity:1" vector-effect="non-scaling-stroke" transform="scale(.41452)" d="M0-63.27c34.925 0 63.27 28.345 63.27 63.27 0 34.925-28.345 63.27-63.27 63.27-34.925 0-63.27-28.345-63.27-63.27 0-34.925 28.345-63.27 63.27-63.27z" />
                                                </g>
                                        </svg>
                                    </div>
                                    <div class="v-subheader _tool-slider-subheader px-1" style="height: 25px;">
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
                                    <svg v-if="spool.runout_lane === ''"
                                         fill="#FF0000"
                                         height="20px"
                                         width="20px"
                                         version="1.1"
                                         id="Capa_1"
                                         style="float: right; margin-top: 2px; margin-left: 5px;"
                                         viewBox="0 0 60 60" xml:space="preserve">
                                    <path d="M43.594,12.5c-9.046,0-16.406,7.851-16.406,17.5c0,6.341-4.836,11.5-10.781,11.5S5.625,36.341,5.625,30
                                        s4.836-11.5,10.781-11.5c2.146,0,4.218,0.67,5.992,1.938c0.593,0.425,1.147,0.911,1.645,1.445c1.061,1.136,2.914,1.14,3.977,0.009
                                        c1.099-1.167,1.103-3.07,0.009-4.243c-0.76-0.813-1.601-1.552-2.499-2.194c-2.704-1.933-5.858-2.954-9.124-2.954
                                        C7.36,12.5,0,20.351,0,30s7.36,17.5,16.406,17.5S32.812,39.649,32.812,30c0-6.341,4.836-11.5,10.781-11.5S54.375,23.659,54.375,30
                                        s-4.836,11.5-10.781,11.5c-2.879,0-5.586-1.195-7.622-3.366c-1.062-1.133-2.915-1.133-3.978,0c-0.531,0.567-0.823,1.32-0.823,2.121
                                        c0,0.802,0.293,1.556,0.824,2.121c3.098,3.305,7.218,5.124,11.599,5.124C52.64,47.5,60,39.649,60,30S52.64,12.5,43.594,12.5z" />
                                    </svg>
                                    <svg v-else="spool.runout_lane === ''"
                                         fill="#00FF00"
                                         height="20px"
                                         width="20px"
                                         version="1.1"
                                         id="Capa_1"
                                         style="float: right; margin-top: 2px;"
                                         viewBox="0 0 60 60" xml:space="preserve">
                                    <path d="M43.594,12.5c-9.046,0-16.406,7.851-16.406,17.5c0,6.341-4.836,11.5-10.781,11.5S5.625,36.341,5.625,30
	                                        s4.836-11.5,10.781-11.5c2.146,0,4.218,0.67,5.992,1.938c0.593,0.425,1.147,0.911,1.645,1.445c1.061,1.136,2.914,1.14,3.977,0.009
	                                        c1.099-1.167,1.103-3.07,0.009-4.243c-0.76-0.813-1.601-1.552-2.499-2.194c-2.704-1.933-5.858-2.954-9.124-2.954
	                                        C7.36,12.5,0,20.351,0,30s7.36,17.5,16.406,17.5S32.812,39.649,32.812,30c0-6.341,4.836-11.5,10.781-11.5S54.375,23.659,54.375,30
	                                        s-4.836,11.5-10.781,11.5c-2.879,0-5.586-1.195-7.622-3.366c-1.062-1.133-2.915-1.133-3.978,0c-0.531,0.567-0.823,1.32-0.823,2.121
	                                        c0,0.802,0.293,1.556,0.824,2.121c3.098,3.305,7.218,5.124,11.599,5.124C52.64,47.5,60,39.649,60,30S52.64,12.5,43.594,12.5z" />
                                    </svg>
                                    <select :name="'run-' + spool.laneName"
                                            class="afclist"
                                            stlye="float: left;"
                                            @change="handleRunoutChange($event, spool)">

                                    <template v-if="spool.runout_lane === ''">
                                    <option :value="''" selected>
                                                NONE
                                            </option>
                                        </template>
                                    <template v-else>
                                    <option :value="''">
                                                NONE
                                            </option>
                                        </template>
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
        <afc-change-spool-dialog
            :show-dialog="showChangeSpoolDialog"
            :index="index"
            :lane-data="selectedLane"
            @close="showChangeSpoolDialog = false"
            @fetch-spool="fetchSpoolData"
        />
    </div>
</template>

<script lang="ts">
import { Component, Mixins } from "vue-property-decorator";
import BaseMixin from "@/components/mixins/base";
import Panel from "@/components/ui/Panel.vue";
import { mdiAdjust, mdiRefresh, mdiDotsVertical } from "@mdi/js";
import AfcChangeSpoolDialog from "@/components/dialogs/AfcChangeSpoolDialog.vue";
import FilamentReelIcon from '@/components/ui/FilamentReelIcon.vue'
import SpoolIcon from "@/components/ui/SpoolIcon.vue";
import { AFCRoot } from '@/store/server/afc'
import BoxTurtleIcon from "@/components/ui/BoxTurtleIcon.vue";

@Component({
    components: {
        Panel,
        AfcChangeSpoolDialog,
        SpoolIcon,
        FilamentReelIcon,
        BoxTurtleIcon
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
                            lanes.push(laneData);
                            laneList.push(laneKey);
                            mapList.push(unit[laneKey]['map']);
                        }
                    }
                }
            }
        }
        this.laneList=laneList.sort()
        this.mapList = mapList.sort()
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
}
</script>

<style scoped>
.afc-panel-wrapper {
    margin-bottom: 24px;
}

    .spool-container {
        display: flex;
        flex-wrap: wrap;
        justify-content: space-evenly;
        gap: 8px;
        padding: 8px;
        padding-top: 0px;
        margin-top: 15px;
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
        max-width: 180px;
        min-width: 80px;
        position: relative;
        cursor: hand;
        transition: box-shadow 0.3s;
        margin-bottom: 8px;
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
    align-items: center;
    gap: 5px;
}

    .afclist {
        background-color: #2e2e2e;
        color: white;
        text-align: right;
    }

.status-wrapper {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 10px;
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
