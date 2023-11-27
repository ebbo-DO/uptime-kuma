<template>
    <transition name="slide-fade" appear>
        <div v-if="monitor">
            <!-- TODO - Translation -->
            <h1 class="text-center">{{ monitor.name }} Report</h1>
            <div class="container">
                <div class="row">
                    <div class="col text-start">
                        <div class="period-options">
                            <button
                                type="button"
                                class="btn btn-primary dropdown-toggle btn-period-toggle"
                                data-bs-toggle="dropdown"
                                aria-expanded="false"
                            >
                                {{ chartPeriodOptions[chartPeriodHrs] }}&nbsp;
                            </button>
                            <ul class="dropdown-menu" style="padding-left: 0">
                                <li
                                    v-for="(item, key) in chartPeriodOptions"
                                    :key="key"
                                >
                                    <a
                                        class="dropdown-item"
                                        :class="{
                                            active: chartPeriodHrs == key,
                                        }"
                                        href="#"
                                        @click="chartPeriodHrs = key"
                                    >
                                        {{ item }}
                                    </a>
                                </li>
                            </ul>
                        </div>
                    </div>
                    <div class="col-6 text-start" v-if="chartPeriodHrs === 'c'">
                        <div class="col-auto">
                            <label class="form-label">
                                <!-- TODO - Translate -->
                                {{ $t("Custom Dates") }}: &nbsp;
                            </label>
                        </div>
                        <Datepicker
                            v-model="this.customDateRange"
                            :dark="$root.isDark"
                            range
                            :partial-range="false"
                            :is-24="false"
                        />
                    </div>
                    <div class="col text-end">
                        <router-link
                            :to="'/dashboard/' + monitor.id"
                            class="btn btn-normal"
                        >
                            <font-awesome-icon icon="chevron-left" />
                            {{ $t("Dashboard") }}
                        </router-link>
                        <button class="btn btn-primary" @click="printReport">
                            <!-- TODO - Translation -->
                            <font-awesome-icon icon="print" /> {{ $t("Print") }}
                        </button>
                    </div>
                </div>
            </div>
            <!-- NOTE - Start Printable Area Here -->
            <div class="printMe">
                <h1 class="text-center">Uptime Report</h1>
                <h3 class="text-center">
                    {{ reportTitle }}
                </h3>
                <Subreport
                    :monitor="monitor"
                    :chartPeriodHrs="chartPeriodHrs"
                    :chartPeriodOptions="chartPeriodOptions"
                />
                <!-- TODO - Translate -->
                <div>{{ $t("Report Generated") }}: {{ reportDate }}</div>
            </div>
        </div>
    </transition>
</template>

<script>
import Subreport from "../components/Subreport.vue";
import Datepicker from "@vuepic/vue-datepicker";
import dayjs from "dayjs";

export default {
    components: {
        Subreport,
        Datepicker,
    },
    data() {
        return {
            chartPeriodHrs: "0",
            chartPeriodOptions: {
                0: this.$t("recent"),
                3: "3h",
                6: "6h",
                24: "24h",
                168: "1w",
                720: "30d",
                2160: "90d",
                8760: "1y",
                lm: "Last Month", //TODO - Translate
                cq: "Current Quarter", //TODO - Translate
                lq: "Last Quarter", //TODO - Translate
                c: "Custom", //TODO - Translate
            },
            customDateRange: [],
        };
    },
    computed: {
        monitor() {
            let id = this.$route.params.id;
            return this.$root.monitorList[id];
        },
        reportTitle() {
            switch (this.chartPeriodHrs) {
                case "0":
                    return this.$t("recent");
                //     3: "3h",
                // 6: "6h",
                // 24: "24h",
                // 168: "1w",
                case "720":
                    // TODO - Translate
                    return `Last 30 ${this.$tc("day", { count: 30 })}`;
                case "2160":
                    // TODO
                    return "90d";
                case "8760":
                    // TODO
                    return "1y";
                case "lm":
                    // TODO
                    return "Last Month";
                case "cq":
                    // TODO
                    return "Current Quarter";
                case "lq":
                    // TODO
                    return "Last Quarter";
                case "c":
                    // TODO
                    return "Custom";
                default:
                    return "";
            }
        },
        reportDate() {
            return dayjs().format("LLLL");
        },
    },
    methods: {
        printReport() {
            var currTitle = document.title;
            document.title = this.monitor.name + " Report";
            //TODO - Add Filters to title?
            window.print();
            document.title = currTitle;
        },
    },
    watch: {
        customDateRange: function (newVal) {
            console.log(newVal);
        },
    },
    created() {
        // Setup Watcher on the root heartbeatList,
        // And mirror latest change to this.heartbeatList
        // this.$watch(
        //     () => this.$root.heartbeatList[this.$route.params.id],
        //     (heartbeatList) => {
        //         // log.debug(
        //         //     "ping_chart",
        //         //     `this.chartPeriodHrs type ${typeof this
        //         //         .chartPeriodHrs}, value: ${this.chartPeriodHrs}`
        //         // );

        //         // eslint-disable-next-line eqeqeq
        //         if (this.chartPeriodHrs != "0") {
        //             const newBeat = heartbeatList.at(-1);

        //             if (
        //                 newBeat &&
        //                 dayjs.utc(newBeat.time) >
        //                     dayjs.utc(this.heartbeatListChart?.at(-1)?.time)
        //             ) {
        //                 this.heartbeatListChart.push(heartbeatList.at(-1));
        //             }
        //         }
        //     },
        //     { deep: true }
        // );

        // Load report period from storage if saved
        let period =
            this.$root.storage()[`report-period-${this.$route.params.id}`];
        // console.log(period);
        if (period != null) {
            if (!isNaN(period)) {
                this.chartPeriodHrs = Math.min(period, 6).toString();
            } else {
                this.chartPeriodHrs = period;
            }
        }
    },
};
</script>

<style lang="scss">
@media print {
    .printMe {
        background-color: white;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 9999;
        width: 100%;
        height: 100%;
        padding: 10px;
    }
    header.d-flex {
        display: none !important;
        height: 0;
    }
    main {
        position: relative;
        height: 0;
    }
    main div.shadow-box.mb-3 {
        display: none;
    }
    div#app {
        // display: none;
        position: relative;
        height: 0;
    }
}
</style>
<style lang="scss" scoped>
@import "../assets/vars.scss";

div {
    .flex {
        display: flex;
    }
}

@media (max-width: 400px) {
    .btn {
        display: inline-flex;
        flex-direction: column;
        align-items: center;
        padding-top: 10px;
        font-size: 0.9em;
    }

    a.btn {
        padding-left: 25px;
        padding-right: 25px;
    }

    .dropdown-clear-data {
        button {
            display: block;
            padding-top: 4px;
        }
    }
}

table {
    font-size: 14px;

    tr {
        transition: all ease-in-out 0.2ms;
    }
}
</style>
