<style lang="scss" scoped>
.btnHomeAxis {
    width: 36px;
    min-width: 36px !important;
}

.btnMinWidthAuto {
    min-width: auto !important;
}

.steps {
    width: 100%;
    > div {
        width: 100%;
        display: flex;
        > button {
            flex-grow: 1;
        }
    }
}
</style>

<template>
    <v-card class="mb-6" v-if="klipperReadyForGui && ['standby', 'paused', 'complete', 'cancelled', 'error'].includes(printer_state)">
        <v-toolbar flat dense>
            <v-toolbar-title>
                <span class="subheading"><v-icon left>mdi-gamepad</v-icon>{{ $t('Panels.ControlPanel.Headline') }}</span>
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-menu :offset-y="true" :close-on-content-click="false" :title="$t('Panels.ControlPanel.SetupControls')" left>
                <template v-slot:activator="{ on, attrs }">
                    <v-btn small class="px-2 minwidth-0" color="grey darken-3" v-bind="attrs" v-on="on"><v-icon small>mdi-cog</v-icon></v-btn>
                </template>
                <v-list>
                    <v-list-item class="minHeight36">
                      <v-checkbox v-model="useCross" class="mt-0" hide-details :label="$t('Panels.ControlPanel.AlternateControls')"></v-checkbox>
                    </v-list-item>
                    <template v-if="useCross">
                        <v-list-item class="minHeight36">
                          <v-checkbox v-model="reverseX" class="mt-0" hide-details :label="$t('Panels.ControlPanel.InvertX')"></v-checkbox>
                        </v-list-item>
                        <v-list-item class="minHeight36">
                          <v-checkbox v-model="reverseY" class="mt-0" hide-details :label="$t('Panels.ControlPanel.InvertY')"></v-checkbox>
                        </v-list-item>
                        <v-list-item class="minHeight36">
                          <v-checkbox v-model="reverseZ" class="mt-0" hide-details :label="$t('Panels.ControlPanel.InvertZ')"></v-checkbox>
                        </v-list-item>
                    </template>
                </v-list>
            </v-menu>
        </v-toolbar>
        <v-container>
            <template v-if="useCross">
                <v-row>
                    <v-col :cols="homeCols">
                        <v-row dense class="mb-1">
                            <v-col cols="3"></v-col>
                            <v-col cols="3">
                                <v-btn class="btnMinWidthAuto fill-width"
                                       :disabled="selectedCrossStep === null || selectedCrossStep === undefined"
                                       @click="doSendMove('Y'+(reverseY ? '-' : '+')+stepsReversed[selectedCrossStep], feedrateXY)"
                                >
                                    <v-icon>mdi-chevron-up</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3"></v-col>
                            <v-col cols="3">
                                <v-btn class="btnMinWidthAuto fill-width"
                                       :disabled="selectedCrossStep === null || selectedCrossStep === undefined"
                                       @click="doSendMove('Z'+(reverseZ ? '-' : '+')+stepsReversed[selectedCrossStep], feedrateZ)"
                                >
                                    <v-icon>mdi-chevron-up</v-icon>
                                </v-btn>
                            </v-col>
                        </v-row>
                        <v-row dense>
                            <v-col cols="3" class="p-rel">
                                <v-btn class="btnMinWidthAuto fill-width p-abs" style="top: -50%; width: calc(100% - 8px);"
                                       :disabled="selectedCrossStep === null || selectedCrossStep === undefined"
                                       @click="doSendMove('X'+(!reverseX ? '-' : '+')+stepsReversed[selectedCrossStep], feedrateXY)"
                                >
                                    <v-icon>mdi-chevron-left</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3">
                                <v-btn class="btnMinWidthAuto fill-width"
                                       :disabled="selectedCrossStep === null || selectedCrossStep === undefined"
                                       @click="doSendMove('Y'+(!reverseY ? '-' : '+')+stepsReversed[selectedCrossStep], feedrateXY)"
                                >
                                    <v-icon>mdi-chevron-down</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3" class="p-rel">
                                <v-btn class="btnMinWidthAuto fill-width p-abs" style="top: -50%; width: calc(100% - 8px);"
                                       :disabled="selectedCrossStep === null || selectedCrossStep === undefined"
                                       @click="doSendMove('X'+(reverseX ? '-' : '+')+stepsReversed[selectedCrossStep], feedrateXY)"
                                >
                                    <v-icon>mdi-chevron-right</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3">
                                <v-btn class="btnMinWidthAuto fill-width"
                                       :disabled="selectedCrossStep === null || selectedCrossStep === undefined"
                                       @click="doSendMove('Z'+(!reverseZ ? '-' : '+')+stepsReversed[selectedCrossStep], feedrateZ)"
                                >
                                    <v-icon>mdi-chevron-down</v-icon>
                                </v-btn>
                            </v-col>
                        </v-row>
                    </v-col>
                    <v-col :cols="homeCols" class="d-flex align-center">
                        <div class="flex-grow-1" style="border-radius: 4px; overflow: hidden;">
                            <v-row dense class="" style="margin-bottom: -2px!important;">
                                <v-col :cols="'quad_gantry_level' in config || 'z_tilt' in config ? 6 : 12">
                                    <v-btn class="w-100"
                                           tile
                                           @click="doHome"
                                           height="30"
                                           :loading="loadings.includes('homeAll')"
                                           :color="homedAxes.includes('xyz') ? 'primary' : 'warning'"
                                    >
                                        <div class="d-flex align-center">
                                            <v-icon>mdi-home</v-icon>
                                            <span class="ml-1">{{ $t('Panels.ControlPanel.ALL') }}</span>
                                        </div>
                                    </v-btn>
                                </v-col>
                                <v-col v-if="'quad_gantry_level' in config || 'z_tilt' in config" cols="6" class="d-flex">
                                    <v-btn v-if="'quad_gantry_level' in config"
                                           class="btnMinWidthAuto flex-grow-1 px-0"
                                           tile
                                           dense
                                           :color="colorQuadGantryLevel"
                                           height="30"
                                           :loading="loadings.includes('qgl')"
                                           @click="doQGL"
                                    >{{ $t('Panels.ControlPanel.QGL') }}</v-btn>
                                    <v-btn v-if="'z_tilt' in config"
                                           class="btnMinWidthAuto flex-grow-1 px-0"
                                           tile
                                           dense
                                           :color="colorZTilt"
                                           height="30"
                                           :loading="loadings.includes('zTilt')"
                                           @click="doZtilt"
                                    >{{ $t('Panels.ControlPanel.ZTilt') }}</v-btn>
                                </v-col>
                            </v-row>
                            <v-row dense class="">
                                <v-col cols="4" class="flex-grow-1">
                                    <v-btn class="btnMinWidthAuto w-100"
                                           tile
                                           height="30"
                                           :loading="loadings.includes('homeX')"
                                           :color="homedAxes.includes('x') ? 'primary' : 'warning'"
                                           @click="doHomeX"
                                    >
                                        {{ $t('Panels.ControlPanel.X') }}
                                    </v-btn>
                                </v-col>
                                <v-col cols="4" class="flex-grow-1">
                                    <v-btn class="btnMinWidthAuto w-100"
                                           tile
                                           height="30"
                                           :loading="loadings.includes('homeY')"
                                           :color="homedAxes.includes('y') ? 'primary' : 'warning'"
                                           @click="doHomeY"
                                    >
                                        {{ $t('Panels.ControlPanel.Y') }}
                                    </v-btn>
                                </v-col>
                                <v-col cols="4" class="flex-grow-1">
                                    <v-btn class="btnMinWidthAuto w-100"
                                           tile
                                           height="30"
                                           :loading="loadings.includes('homeZ')"
                                           :color="homedAxes.includes('z') ? 'primary' : 'warning'"
                                           @click="doHomeZ"
                                    >
                                        {{ $t('Panels.ControlPanel.Z') }}
                                    </v-btn>
                                </v-col>
                            </v-row>
                        </div>
                    </v-col>
                </v-row>
                <v-row no-gutters class="mt-3">
                    <v-col class="col-12">
                        <v-btn-toggle v-if="stepsReversed.length > 0" dense no-gutters style="flex-wrap: nowrap; width: 100%;" v-model="selectedCrossStep">
                            <v-btn dense class="btnMinWidthAuto flex-grow-1 px-0" v-for="steps of stepsReversed" :key="'all-'+steps">
                                <span class="body-2">{{ steps }}</span>
                            </v-btn>
                        </v-btn-toggle>
                        <div class="font-weight-bold warning rounded pa-2" v-else>
                             {{ $t('Panels.ControlPanel.PleaseConfigureSteps') }}<br>
                            <router-link style="color: white;" to="/settings/interface">{{ $t('Panels.ControlPanel.SettingsInterfaceControl') }}</router-link>
                        </div>
                    </v-col>
                </v-row>
            </template>
            <template v-else>
                <v-row no-gutters>
                    <v-col class="col-12  pb-0 text-center">
                        <v-btn small @click="doHome" :loading="loadings.includes('homeAll')" :color="homedAxes.includes('xyz') ? 'primary' : 'warning'"><v-icon class="mr-1">mdi-home</v-icon>{{ $t('Panels.ControlPanel.ALL') }}</v-btn>
                        <v-btn small @click="doQGL" :loading="loadings.includes('qgl')" :color="colorQuadGantryLevel" class="ml-2" v-if="'quad_gantry_level' in config">{{ $t('Panels.ControlPanel.QGL') }}</v-btn>
                        <v-btn small @click="doZtilt" :loading="loadings.includes('zTilt')" :color="colorZTilt" class="ml-2" v-if="'z_tilt' in config">{{ $t('Panels.ControlPanel.ZTilt') }}</v-btn>
                    </v-col>
                </v-row>
                <v-row no-gutters class="mt-2">
                    <v-col class="text-center">
                        <v-btn-toggle dense no-gutters class="row no-gutters justify-center" style="flex-wrap: nowrap; width: 100%;" >
                            <v-btn @click="doSendMove('X-'+steps, feedrateXY)" class="btnMinWidthAuto col" v-for="steps of stepsXYsorted" v-bind:key="'x-'+steps"><span class="body-2">–{{ steps }}</span></v-btn>
                            <v-btn @click="doHomeX" :color="homedAxes.includes('x') ? 'primary' : 'warning'" :loading="loadings.includes('homeX')" class="font-weight-bold btnHomeAxis">{{ $t('Panels.ControlPanel.X') }}</v-btn>
                            <v-btn @click="doSendMove('X+'+steps, feedrateXY)" class="btnMinWidthAuto col" v-for="steps of stepsXYsortedReverse" v-bind:key="'x+'+steps"><span class="body-2">+{{ steps }}</span></v-btn>
                        </v-btn-toggle>
                    </v-col>
                </v-row>
                <v-row no-gutters class="mt-3">
                    <v-col class="text-center">
                        <v-btn-toggle dense no-gutters class="row no-gutters justify-center" style="flex-wrap: nowrap; width: 100%;" >
                            <v-btn @click="doSendMove('Y-'+steps, feedrateXY)" class="btnMinWidthAuto col" v-for="steps of stepsXYsorted" v-bind:key="'y-'+steps"><span class="body-2">–{{ steps }}</span></v-btn>
                            <v-btn @click="doHomeY" :color="homedAxes.includes('y') ? 'primary' : 'warning'" :loading="loadings.includes('homeY')" class="font-weight-bold btnHomeAxis">{{ $t('Panels.ControlPanel.Y') }}</v-btn>
                            <v-btn @click="doSendMove('Y+'+steps, feedrateXY)" class="btnMinWidthAuto col" v-for="steps of stepsXYsortedReverse" v-bind:key="'y+'+steps"><span class="body-2">+{{ steps }}</span></v-btn>
                        </v-btn-toggle>
                    </v-col>
                </v-row>
                <v-row no-gutters class="mt-3">
                    <v-col class="text-center">
                        <v-btn-toggle dense no-gutters class="row no-gutters justify-center" style="flex-wrap: nowrap; width: 100%;" >
                            <v-btn @click="doSendMove('Z-'+steps, feedrateZ)" class="btnMinWidthAuto col" v-for="steps of stepsZsorted" v-bind:key="'z-'+steps"><span class="body-2">–{{ steps }}</span></v-btn>
                            <v-btn @click="doHomeZ" :color="homedAxes.includes('z') ? 'primary' : 'warning'" :loading="loadings.includes('homeZ')" class="font-weight-bold btnHomeAxis">{{ $t('Panels.ControlPanel.Z') }}</v-btn>
                            <v-btn @click="doSendMove('Z+'+steps, feedrateZ)" class="btnMinWidthAuto col" v-for="steps of stepsZsortedReverse" v-bind:key="'z+'+steps"><span class="body-2">+{{ steps }}</span></v-btn>
                        </v-btn-toggle>
                    </v-col>
                </v-row>
            </template>
            <template v-if="existsExtruder">
                <v-row>
                    <v-col class="pa-0">
                        <v-divider></v-divider>
                    </v-col>
                </v-row>
                <v-row class="">
                    <v-col class="col col-md-6 pt-2">
                        <span class="text--disabled" style="font-size: .9em">{{ $t("Panels.ControlPanel.FeedAmountIn") }} [mm]</span>
                        <v-btn-toggle class="mt-1" dense no-gutters style="flex-wrap: nowrap; width: 100%;" >
                            <v-btn v-for="amount in feedamountsSorted" v-bind:key="amount" @click="setFeedAmount(amount)" dense :class="(amount === currentFeedAmount ? 'v-btn--active' : '') + ' btnMinWidthAuto flex-grow-1 px-0 _btnFeedrate'">{{ amount }}</v-btn>
                        </v-btn-toggle>
                    </v-col>
                    <v-col class="col col-md-6 pt-2">
                        <span class="text--disabled" style="font-size: .9em">{{ $t("Panels.ControlPanel.FeedrateIn") }} [mm/s]</span>
                        <v-btn-toggle class="mt-1" dense no-gutters style="flex-wrap: nowrap; width: 100%;" >
                            <v-btn v-for="rate in feedratesSorted" v-bind:key="rate" @click="setFeedrate(rate)" dense :class="(rate === currentFeedRate ? 'v-btn--active' : '') + ' btnMinWidthAuto flex-grow-1 px-0 _btnFeedrate'">{{ rate }}</v-btn>
                        </v-btn-toggle>
                    </v-col>
                </v-row>
                <v-row class="">
                    <v-col class="col text-center pt-0">
                        <v-btn small @click="sendRetract()" class="mx-3" :loading="loadings.includes('btnRetract')" :disabled="!boolExtrudePossible"><v-icon small class="mr-1">mdi-arrow-up-bold</v-icon> {{ $t('Panels.ControlPanel.Retract') }}</v-btn>
                        <v-btn small @click="sendDetract()" class="mx-3" :loading="loadings.includes('btnDetract')" :disabled="!boolExtrudePossible"><v-icon small class="mr-1">mdi-arrow-down-bold</v-icon> {{ $t('Panels.ControlPanel.Extrude') }}</v-btn>
                    </v-col>
                </v-row>
            </template>
        </v-container>
    </v-card>
</template>

<script lang="ts">
import {Component, Mixins} from "vue-property-decorator";
import BaseMixin from "../mixins/base";

@Component
export default class ControlPanel extends Mixins(BaseMixin) {
    private crossWidth = 40
    private crossHeight = 40
    private homeCols = 6

    get homedAxes(): string {
        return this.$store.state.printer?.toolhead?.homed_axes ?? ""
    }

    get config() {
        return this.$store.state.printer?.configfile?.settings ?? {}
    }

    get absolute_coordinates() {
        return this.$store.state.printer?.gcode_move?.absolute_coordinates ?? true
    }

    get feedamounts() {
        return this.$store.state.gui.dashboard?.extruder?.feedamounts ?? []
    }

    get feedrates() {
        return this.$store.state.gui.dashboard?.extruder?.feedrates ?? []
    }

    get feedrateXY() {
        return this.$store.state.gui.dashboard?.control?.feedrateXY ?? 100
    }

    get stepsXY() {
        return this.$store.state.gui.dashboard?.control?.stepsXY ?? []
    }

    get stepsXYsorted() {
        return [...this.$store.state.gui.dashboard.control.stepsXY].sort(function(a, b) { return b-a })
    }

    get stepsXYsortedReverse() {
        return [...this.$store.state.gui.dashboard.control.stepsXY].sort(function(a, b) { return a-b })
    }

    get feedrateZ() {
        return this.$store.state.gui.dashboard?.control?.feedrateZ ?? 10
    }

    get stepsZ() {
        return this.$store.state.gui.dashboard?.control?.stepsZ ?? []
    }

    get stepsZsorted() {
        return [...this.$store.state.gui.dashboard.control.stepsZ].sort(function(a, b) { return b-a })
    }

    get stepsZsortedReverse() {
        return [...this.$store.state.gui.dashboard.control.stepsZ].sort(function(a, b) { return a-b })
    }

    get stepsAll() {
        return this.$store.state.gui.dashboard?.control?.stepsAll ?? []
    }

    get steps() {
        return Array.from(new Set([
            ...(this.stepsAll ?? [])
        ])).sort((a, b) => b-a)
    }

    get stepsReversed() {
        return Array.from(new Set([
            ...(this.stepsAll ?? []),
        ])).sort((a, b) => a-b)
    }

    get reverseX() {
        return this.$store.state.gui.dashboard.control.reverseX
    }

    set reverseX(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: "dashboard.control.reverseX", value: newVal })
    }

    get reverseY() {
        return this.$store.state.gui.dashboard.control.reverseY
    }

    set reverseY(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: "dashboard.control.reverseY", value: newVal })
    }

    get reverseZ() {
        return this.$store.state.gui.dashboard.control.reverseZ
    }

    set reverseZ(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: "dashboard.control.reverseZ", value: newVal })
    }

    get useCross() {
        return this.$store.state.gui.dashboard.control.useCross
    }

    set useCross(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: "dashboard.control.useCross", value: newVal })
    }

    get selectedCrossStep() {
        return this.$store.state.gui.dashboard.control.selectedCrossStep
    }

    set selectedCrossStep(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: "dashboard.control.selectedCrossStep", value: newVal })
    }

    get existsExtruder() {
        return 'extruder' in this.$store.state.printer
    }

    get feedamountsSorted() {
        return [...this.feedamounts].sort((a,b) => { return b-a })
    }

    get feedratesSorted () {
        return [...this.feedrates].sort((a,b) => { return b-a })
    }
    get currentFeedAmount() {
        return parseFloat(this.$store.state.gui.dashboard.extruder.feedamount)
    }

    set currentFeedAmount(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: "dashboard.extruder.feedamount", value: newVal })
    }

    get currentFeedRate() {
        return parseFloat(this.$store.state.gui.dashboard.extruder.feedrate)
    }

    set currentFeedRate(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: "dashboard.extruder.feedrate", value: newVal })
    }

    get colorQuadGantryLevel() {
        const status = this.$store.state.printer.quad_gantry_level?.applied ?? true

        return status ? "primary" : "warning"
    }

    get colorZTilt() {
        const status = this.$store.state.printer.z_tilt?.applied ?? true

        return status ? "primary" : "warning"
    }

    get boolExtrudePossible() {
        return this.$store.getters['printer/getExtrudePossible']
    }

    doHome() {
        this.$store.dispatch('server/addEvent', { message: "G28", type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: "G28" }, { loading: 'homeAll' })
    }
    
    doHomeX() {
        this.$store.dispatch('server/addEvent', { message: "G28 X", type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: "G28 X" }, { loading: 'homeX' })
    }
    
    doHomeY() {
        this.$store.dispatch('server/addEvent', { message: "G28 Y", type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: "G28 Y" }, { loading: 'homeY' })
    }

    doHomeZ() {
        this.$store.dispatch('server/addEvent', { message: "G28 Z", type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: "G28 Z" }, { loading: 'homeZ' })
    }

    doQGL() {
        this.$store.dispatch('server/addEvent', { message: "QUAD_GANTRY_LEVEL", type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: "QUAD_GANTRY_LEVEL" }, { loading: 'qgl' })
    }

    doZtilt() {
        this.$store.dispatch('server/addEvent', { message: "Z_TILT_ADJUST", type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: "Z_TILT_ADJUST" }, { loading: 'zTilt' })
    }

    doSendMove(gcode: string, feedrate: number) {
        if (this.absolute_coordinates) {
            gcode = "G91" + "\n" +
                "G1 " + gcode + " F"+feedrate*60 + "\n" +
                "G90"
        } else gcode = "G1 " + gcode + " F"+feedrate*60

        this.doSend(gcode)
    }

    doSend(gcode: string) {
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode })
    }

    setFeedAmount(value: number) {
        this.currentFeedAmount = value
    }

    setFeedrate(value: number) {
        this.currentFeedRate = value
    }

    sendRetract() {
        const gcode = "M83\nG1 E-"+this.currentFeedAmount+" F"+(this.currentFeedRate * 60)
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode }, { loading: 'btnRetract' })
    }

    sendDetract() {
        const gcode = "M83\nG1 E"+this.currentFeedAmount+" F"+(this.currentFeedRate * 60)
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode }, { loading: 'btnDetract' })
    }

    onResize() {
        this.homeCols = window.screen.width < 360 ? 12 : 6;
    }

    created() {
        window.addEventListener('resize', this.onResize);
    }

    mounted() {
        if (window.screen.width < 330) {
            this.homeCols = 12;
        }
    }

    destroyed() {
        window.removeEventListener('resize', this.onResize);
    }
}
</script>
