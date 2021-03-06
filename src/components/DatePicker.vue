<template>
    <div class="date-picker">
        <header class="date-picker__header">
            <!-- Arrow - prev month -->
            <button @click="prevMonth" class="date-picker__select-month">
                <i class="arrow arrow--left"></i>
            </button>
            <p class="date-picker__title">
                {{ currentMonth }}, {{ currentYear }}
            </p>
            <!-- Arrow - next month -->
            <button @click="nextMonth" class="date-picker__select-month">
                <i class="arrow arrow--right"></i>
            </button>
        </header>
        <main class="date-picker__content">
            <section class="calendar">
                <header class="calendar__day-titles">
                    <span class="day-title"
                          v-for="day in weekdays"
                          :key="day"
                    >{{ day }}</span>
                </header>
                <main class="calendar__days">
                    <div class="day"
                         v-for="day in currentCalendar"
                         :key="day.date"
                         :class="[isThisMonth(day.date), isToday(day.date), isSelected(day), isFirst(day), isLast(day)]"
                         @click="selectDay(day.date)"
                    >
                        <p class="day__content">{{ getDay( day.date ) }}</p>
                    </div>
                </main>
                <footer class="date-picker__footer">
                    <span class="select-date__title">Select range:</span>
                    <input disabled :value="getFormattedDate()[0]" type="text" class="select-date" name="date-from">
                    <input disabled :value="getFormattedDate()[1]" type="text" class="select-date" name="date-to">
                </footer>
            </section>
        </main>


        <!-- Test -->
    </div>
</template>

<script>
    export default {
        name: "DatePicker",
        props: {
            highlightToday: {
                type: Boolean,
                default: false
            },
            range: {
                type: Boolean,
                default: false
            },
            formatted: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                today: new Date(),
                date: new Date(),
                inputDate: [],
                selectClick: 0
            };
        },
        methods: {
            nextMonth() {
                this.date = new Date( this.date.setMonth( this.date.getMonth() + 1 ) );
                this.highlightSelectedDates();
            },
            prevMonth() {
                this.date = new Date( this.date.setMonth( this.date.getMonth() - 1 ) );
                this.highlightSelectedDates();
            },
            getNumberOfDaysInMonth(date) {
                return 32 - new Date( date.getFullYear(), date.getMonth(), 32 ).getDate();
            },
            getFirstDayOfMonth(date) {
                return ( new Date( date.getFullYear(), date.getMonth() ).getDay() );
            },
            getDaysFromPrevMonth(numOfDays) {
                let prevMonth = this.getDates.prevMonth;
                let dates = [];

                for( let i = 0; i < numOfDays; i++ ) {
                    dates.push( prevMonth[ prevMonth.length - numOfDays + i ] );
                }
                return dates;
            },
            getDaysFromNextMonth(numOfDays) {
                let nextMonth = this.getDates.nextMonth;

                let dates = [];

                for( let i = 0; i < numOfDays; i++ ) {
                    dates.push( nextMonth[ i ] );
                }

                return dates;
            },
            getDay(date) {
                return new Date( date ).getDate();
            },
            isThisMonth(date) {
                let currentMonth = new Date( this.date ).getMonth();
                return new Date( date ).getMonth() === currentMonth ? "day__this-month" : "";
            },
            isToday(date) {
                if( !this.$props.highlightToday ) {
                    return false;
                }
                let today = this.today;
                let providedDate = new Date( date );

                if(
                    today.getDate() === providedDate.getDate() &&
                    today.getMonth() === providedDate.getMonth() &&
                    today.getFullYear() === providedDate.getFullYear()
                ) {
                    return "day__today";
                }
                return "";
            },
            isSelected(dateObject) {
                return ( dateObject.selected ) ? "day__highlight" : "";
            },
            isFirst(dateObject) {
                return ( dateObject.first ) ? "day__highlight--first" : "";
            },
            isLast(dateObject) {
                return ( dateObject.last ) ? "day__highlight--last" : "";
            },
            compareDates( dates ) {
                let formattedDates = [];

                dates.forEach(date => {
                    formattedDates.push(+new Date(date));
                });

                formattedDates.sort((a, b) => a - b);

                return formattedDates;
            },
            formattedDate(date) {
                let d = new Date( date );

                let month = "" + ( d.getMonth() + 1 );
                let year = d.getFullYear();
                let day = "" + d.getDate();

                month = month.length < 2 ? "0" + month : month;
                day = day.length < 2 ? "0" + day : day;

                return [ year, month, day ].join("-");
            },
            selectDay(data) {
                if( !this.selectClick ) {
                    this.selectClick = 1;
                    return this.$set( this.inputDate, 0, data );
                }
                this.selectClick = 0;
                return this.$set( this.inputDate, 1, data );
            },
            highlightSelectedDates() {
                let calendar = this.currentCalendar;

                let start = this.inputDate[0];
                let end = this.inputDate[1];

                let indexes = Object.entries(calendar).map( (dateObjects, index) => {
                    let dateObject = dateObjects[1];
                    return ( dateObject.date === start || dateObject.date === end ) ? index : null;
                } ).filter( value => value );

                this.currentCalendar.forEach( (value, index) => {
                    value.selected = false;
                    value.first = false;
                    value.last = false;

                    if( !indexes[1] && index === indexes[0] ) {
                        value.selected = true;
                        value.first = true;
                        value.last = true;
                        return value;
                    }
                    else if( indexes[0] && indexes[1] ) {
                        value.selected = true;

                        if( index === indexes[0] ) {
                            value.first = true;
                        }

                        if( index === indexes[1] ) {
                            value.last = true;
                        }

                        if( index < indexes[0] || index > indexes[1] ) {
                            value.selected = false;
                        }

                        return value;
                    }

                    return value;
                } );
            },
            setFormattedDate(date) {
                let dateArray = date.split( "-" );
                let formattedDate = [ dateArray[2], dateArray[1], dateArray[0]].join("-");

                if( this.$props.range ) {
                    let dateCopy = new Date( formattedDate );
                    let nextDay = new Date( dateCopy.setDate( dateCopy.getDate() + 1 ) );

                    this.$set( this.inputDate, 0, formattedDate );
                    this.$set( this.inputDate, 1, this.formattedDate(nextDay) );
                    return;
                }

                this.selectClick = 1;
                this.$set( this.inputDate, 0, formattedDate );
            },
            getFormattedDate() {
                let dates = this.inputDate;

                return dates.map( date => {
                    return date.split("-").reverse().join("-");
                } );
            }
        },
        computed: {
            currentYear() {
                return this.date.getFullYear();
            },
            currentMonth() {
                let month = this.date.toLocaleString("default", { month: "long" });
                return month.charAt(0).toUpperCase() + month.slice(1);
            },
            weekdays() {
                let dates = [];
                let firstDay = this.getFirstDayOfMonth( this.date );
                let firstDayName = new Date( this.date.getFullYear(), this.date.getMonth() ).toLocaleString("default", { weekday: "long" }).substring(0,2);


                for( let i = 1; i <= 7; i++ ) {
                    let date = new Date( this.today.getFullYear(), this.today.getMonth(), i );
                    dates.push( date.toLocaleString("default", { weekday: "long" }).substring(0,2) );
                }

                dates.map( date => {
                    if( dates.indexOf( firstDayName ) !== firstDay - 1 ) {
                        dates.push( dates.shift() );
                    }
                    return date;
                } );

                return dates;
            },
            getDates() {
                let date = new Date( this.date );

                let months = {
                    thisMonth: new Date( date ),
                    prevMonth: new Date( date.setMonth( date.getMonth() - 1 ) ),
                    nextMonth: new Date( date.setMonth( date.getMonth() + 2 ) )
                };

                let numberOfDays = {
                    thisMonth: this.getNumberOfDaysInMonth( months.thisMonth ),
                    prevMonth: this.getNumberOfDaysInMonth( months.prevMonth ),
                    nextMonth: this.getNumberOfDaysInMonth( months.nextMonth )
                };

                let dates = {
                    thisMonth: [],
                    prevMonth: [],
                    nextMonth: []
                };

                Object.entries(numberOfDays).forEach( entry => {
                    for( let i = 0; i < entry[1]; i++ ) {
                        let day = i + 1;

                        let date = new Date( months[entry[0]].getFullYear(), months[entry[0]].getMonth(), day );
                        dates[entry[0]].push( this.formattedDate( date ) );
                    }
                } );

                return dates;
            },
            currentCalendar() {
                const fields = 42;

                let numberOfDaysInMonth = this.getNumberOfDaysInMonth( this.date );
                let firstDayInMonth = this.getFirstDayOfMonth( this.date );
                let currentMonthDays = this.getDates.thisMonth;

                let calendar = [];
                let date = 0;

                let prevMonthDays = 0;
                let nextMonthDays = 0;

                for( let i = 1; i <= fields; i++ ) {

                    if( i < firstDayInMonth ) {
                        calendar.push( 0 );
                        prevMonthDays++;
                    }
                    else if( date >= numberOfDaysInMonth ) {
                        calendar.push( 0 );
                        nextMonthDays++;
                    }
                    else {
                        calendar.push( currentMonthDays[ date ] );
                        date++;
                    }

                }

                let prevMonthDaysToFill = this.getDaysFromPrevMonth( prevMonthDays );
                let nextMonthDaysToFill = this.getDaysFromNextMonth( nextMonthDays ).reverse();

                for( let i = 0; i < prevMonthDaysToFill.length; i++ ) {
                    calendar[i] = prevMonthDaysToFill[i];
                }

                for( let i = 0; i < nextMonthDays; i++ ) {
                    calendar[ calendar.length - 1 - i ] = nextMonthDaysToFill[i];
                }

                calendar = calendar.map( value => {
                    return {
                        date: value,
                        selected: false,
                    };
                } );

                // console.log( calendar );

                return calendar;
            }
        },
        watch: {
            inputDate( newVal ) {
                let formattedDates = this.compareDates( newVal );
                formattedDates = formattedDates.map( date => {
                    return this.formattedDate( date );
                } );

                this.inputDate[0] = formattedDates[0];
                this.inputDate[1] = formattedDates[1];

                this.highlightSelectedDates();
            }
        },
        mounted() {
            let bindValue = this.$attrs.value;

            if( this.$props.range && !this.$props.formatted ) {
                bindValue = bindValue.map( value => this.formattedDate( value ) );
                this.$set( this.inputDate, 0, bindValue[0] );
                this.$set( this.inputDate, 1, bindValue[1] );
                return;
            }

            if( this.$props.formatted ) {
                this.setFormattedDate( bindValue );
                return;
            }

            let formattedDate = this.formattedDate( bindValue );
            this.setFormattedDate( formattedDate );
        }
    };
</script>

<style lang="scss" scoped>
    // Variables
    $primaryColor: #00B2F2;
    $highlightColor: #CCF1FF;
    $headingColor: rgba( 0,0,0,.15 );
    $bodyPadding: 30px;
    $padding: 10px;

    .arrow {
        border: solid #666666;
        border-width: 0 3px 3px 0;
        display: inline-block;
        padding: 3px;

        &--right {
            transform: rotate(-45deg);
            -webkit-transform: rotate(-45deg);
        }
        &--left {
            transform: rotate(135deg);
            -webkit-transform: rotate(135deg);
        }
    }

    .date-picker {
        background-color: #FFF;
        padding: $bodyPadding;
        mix-blend-mode: normal;


        &__header {
            margin-bottom: 32px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
        }

        &__select-month {
            background-color: transparent;
            border: none;
            outline: none;
            padding: 8px;
            cursor: pointer;
            flex: 0 1 auto;
        }

        &__title {
            color: #333333;
            font-weight: bold;
            display: inline-block;
            flex: 1 1 auto;
        }

        &__content {
            display: block;
            margin-bottom: 24px;
        }

        &__footer {
            display: flex;
            align-items: center;
        }
    }

    .calendar {

        &__day-titles {
            padding: 16px 0;
            display: flex;
            justify-content: space-around;
            border-bottom: 1px solid rgba(0, 0, 0, 0.15);

            .day-title {
                color: $headingColor;
                width: calc(100% / 7);
                text-transform: capitalize;
            }
        }

        &__days {
            display: flex;
            justify-content: flex-start;
            flex-wrap: wrap;
            flex: 1 1 100%;

            & .day {
                width: calc( 100% / 7 );
                text-align: center;
                cursor: pointer;
                position: relative;
                color: lighten( #333333, 60 );
                margin: 2px 0;

                &:before {
                    display: block;
                    content: "";
                    width: 100%;
                    padding-top: 100%;
                }

                &__content {
                    position: absolute;
                    top: 0;
                    left: 0;
                    bottom: 0;
                    right: 0;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                }

                &__today {
                    background-color: $primaryColor;
                    border-radius: 100%;
                    color: #FFFFFF;
                    font-weight: bold;
                }

                &__this-month {
                    color: #333333;
                }

                &__highlight {
                    background-color: $highlightColor;
                    border-radius: 0;

                    &--first {
                        border-top-left-radius: 100%;
                        border-bottom-left-radius: 100%;
                    }
                    &--last {
                        border-top-right-radius: 100%;
                        border-bottom-right-radius: 100%;
                    }
                }
            }
        }

        .select-date {
            padding: 12px;
            text-align: center;
            font-family: Avenir, Helvetica, Arial, sans-serif;
            border: 1px solid #DAE2E6;
            border-radius: 6px;
            outline: none;

            &:first-of-type {
                margin-right: 16px;
            }

            &:focus {
                border: 1px solid darken( #DAE2E6, 10 );
            }

            &__title {
                font-size: 12px;
                color: #666666;
                margin-right: 8px;
            }
        }
    }
</style>