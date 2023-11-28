<template>
    <transition name="slide-fade" appear>
        <div v-if="monitor">
            <h1 class="text-center">{{ monitor.name }} {{ $t("Report") }}</h1>
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
                            <ul class="dropdown-menu" style="padding-left: 0;">
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
                    <div v-if="chartPeriodHrs === 'c'" class="col-6 text-start">
                        <div class="col-auto">
                            <label class="form-label">
                                {{ $t("Custom Dates") }}: &nbsp;
                            </label>
                        </div>
                        <Datepicker
                            v-model="customDateRange"
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
                            <font-awesome-icon icon="print" /> {{ $t("Print") }}
                        </button>
                    </div>
                </div>
            </div>
            <!-- NOTE - Start Printable Area Here -->
            <div class="printMe">
                <h1 class="text-center">Uptime Report</h1>
                <!-- eslint-disable-next-line vue/no-v-html -->
                <h3 class="text-center" v-html="reportTitle"></h3>
                <!-- <Subreport
                    :monitor="monitor"
                    :chartPeriodHrs="chartPeriodHrs"
                    :chartPeriodOptions="chartPeriodOptions"
                /> -->
                <Subreport
                    :monitor="monitor"
                    :chartPeriodHrs="chartPeriodHrs"
                    :chartPeriodOptions="chartPeriodOptions"
                    :lastMonthStart="lastMonthStart"
                    :lastMonthEnd="lastMonthEnd"
                    :currentQuarterStart="currentQuarterStart"
                    :currentQuarterEnd="currentQuarterEnd"
                    :previousQuarterStart="previousQuarterStart"
                    :previousQuarterEnd="previousQuarterEnd"
                    :customDateRange="customDateRange"
                />
                <div>{{ $t("Report Generated") }}: {{ reportDateDisplay }}</div>
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
            chartPeriodHrs: 0,
            chartPeriodOptions: {
                0: this.$t("recent"),
                3: "3h",
                6: "6h",
                24: "24h",
                168: "7d",
                720: "30d",
                2160: "90d",
                8760: "1y",
                lm: this.$t("Previous Month"),
                cq: this.$t("Current Quarter"),
                lq: this.$t("Previous Quarter"),
                c: this.$t("Custom"),
            },
            customDateRange: [],
            lastMonthStart: dayjs().subtract(1, "month").startOf("month"),
            lastMonthEnd: dayjs().subtract(1, "month").endOf("month"),
            currentQuarterStart: dayjs().startOf("quarter"),
            currentQuarterEnd: dayjs().endOf("quarter"),
            previousQuarterStart: dayjs()
                .subtract(1, "quarter")
                .startOf("quarter"),
            previousQuarterEnd: dayjs().subtract(1, "quarter").endOf("quarter"),
            reportDate: dayjs().format("LLLL"),
        };
    },
    computed: {
        reportDateDisplay() {
            return this.reportDate;
        },
        lastMonthStartDisplay() {
            return this.lastMonthStart.format("LLLL");
        },
        lastMonthEndDisplay() {
            return this.lastMonthEnd.format("LLLL");
        },
        currentQuarterStartDisplay() {
            return this.currentQuarterStart.format("LLLL");
        },
        currentQuarterEndDisplay() {
            return this.currentQuarterEnd.format("LLLL");
        },
        previousQuarterStartDisplay() {
            return this.previousQuarterStart.format("LLLL");
        },
        previousQuarterEndDisplay() {
            return this.previousQuarterEnd.format("LLLL");
        },
        customDateDisplay() {
            // console.log(this.customDateRange);
            if (this.customDateRange.length > 0) {
                return `${dayjs(this.customDateRange[0]).format(
                    "LLLL",
                )} - ${dayjs(this.customDateRange[1]).format("LLLL")}`;
            }
            return "";
        },
        monitor() {
            let id = this.$route.params.id;
            return this.$root.monitorList[id];
        },
        reportTitle() {
            switch (this.chartPeriodHrs) {
                case "0":
                    return this.$t("recent");
                case "3":
                    return `${this.$t("Last")} 3 ${this.$tc("hour", {
                        count: 3,
                    })}`;
                case "6":
                    return `${this.$t("Last")} 6 ${this.$tc("hour", {
                        count: 6,
                    })}`;
                case "24":
                    return `${this.$t("Last")} 24 ${this.$tc("hour", {
                        count: 24,
                    })}`;
                case "168":
                    return `${this.$t("Last")} 7 ${this.$tc("day", {
                        count: 7,
                    })}`;
                case "720":
                    return `${this.$t("Last")} 30 ${this.$tc("day", {
                        count: 30,
                    })}`;
                case "2160":
                    return `${this.$t("Last")} 90 ${this.$tc("day", {
                        count: 90,
                    })}`;
                case "8760":
                    return `${this.$t("Last")} 1 ${this.$t("year")}`;
                case "lm":
                    return `${this.$t("Previous Month")}<br/>${
                        this.lastMonthStartDisplay
                    } - ${this.lastMonthEndDisplay}`;
                case "cq":
                    return `${this.$t("Current Quarter")}<br/>${
                        this.currentQuarterStartDisplay
                    } - ${this.currentQuarterEndDisplay}`;
                case "lq":
                    return `${this.$t("Previous Quarter")}<br/>${
                        this.previousQuarterStartDisplay
                    } - ${this.previousQuarterEndDisplay}`;
                case "c":
                    return `${this.$t("Custom")}<br/>${this.customDateDisplay}`;
                default:
                    return "";
            }
        },
    },
    watch: {
        chartPeriodHrs: function (newVal) {
            this.updateReportDate();
            switch (newVal) {
                case "lm":
                    this.updateLastMonthStart();
                    this.updateLastMonthEnd();
                    break;
                case "cq":
                    this.updateCurrentQuarterStart();
                    this.updateCurrentQuarterEnd();
                    break;
                case "lq":
                    this.updatePreviousQuarterStart();
                    this.updatePreviousQuarterEnd();
                    break;
                // case "c":
                //     return `${this.$t("Custom")}`;
                default:
                    break;
            }
        },
        // customDateRange: function (newDates) {
        //     console.log(newDates);
        //     // if(newDates)
        // },
    },
    created() {
        // Load report period from storage if saved
        let period =
            this.$root.storage()[`report-period-${this.$route.params.id}`];
        if (period != null) {
            if (!isNaN(period)) {
                this.chartPeriodHrs = Math.min(period, 6);
            } else {
                this.chartPeriodHrs = period;
            }
        }
    },
    methods: {
        updateReportDate() {
            this.reportDate = dayjs().format("LLLL");
        },
        printReport() {
            const currTitle = document.title;
            document.title = `${
                this.monitor.name
            } Uptime Report ${this.reportTitle.replace("<br/>", " ")}`;
            window.print();
            document.title = currTitle;
        },
        updateLastMonthStart() {
            this.lastMonthStart = dayjs().subtract(1, "month").startOf("month");
        },
        updateLastMonthEnd() {
            this.lastMonthEnd = dayjs().subtract(1, "month").endOf("month");
        },
        updateCurrentQuarterStart() {
            this.currentQuarterStart = dayjs().startOf("quarter");
        },
        updateCurrentQuarterEnd() {
            this.currentQuarterEnd = dayjs().endOf("quarter");
        },
        updatePreviousQuarterStart() {
            this.previousQuarterStart = dayjs()
                .subtract(1, "quarter")
                .startOf("quarter");
        },
        updatePreviousQuarterEnd() {
            this.previousQuarterEnd = dayjs()
                .subtract(1, "quarter")
                .endOf("quarter");
        },
    },
};
</script>

<style lang="scss">
@media print {
    .print-me {
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
