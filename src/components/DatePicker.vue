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
                    <span class="day-title">Po</span>
                    <span class="day-title">Ut</span>
                    <span class="day-title">St</span>
                    <span class="day-title">Št</span>
                    <span class="day-title">Pi</span>
                    <span class="day-title">So</span>
                    <span class="day-title">Ne</span>
                </header>
                <main class="calendar__days">
                    <div class="day"
                         v-for="day in currentCalendar"
                         :key="day"
                         :data-date="day"
                         :class="[isThisMonth(day), isToday(day)]"
                         @click="selectDay"
                    >
                        <p class="day__content">{{ getDay( day ) }}</p>
                    </div>
                </main>
                <footer class="date-picker__footer">
                    <span class="select-date__title">Select range:</span>
                    <input v-model="inputDate[0]" type="date" class="select-date" name="date-from">
                    <input v-model="inputDate[1]" type="date" class="select-date" name="date-to">
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
            },
            prevMonth() {
                this.date = new Date( this.date.setMonth( this.date.getMonth() - 1 ) );
            },
            getNumberOfDaysInMonth(date) {
                return 32 - new Date( date.getFullYear(), date.getMonth(), 32 ).getDate();
            },
            getFirstDayOfMonth(date) {
                return ( new Date( date.getFullYear(), date.getMonth() ).getDay() );
            },
            getDaysFromPrevMonth(numOfDays) {
                let prevMonth = this.prevMonthDates;
                let dates = [];

                for( let i = 0; i < numOfDays; i++ ) {
                    dates.push( prevMonth[ prevMonth.length - numOfDays + i ] );
                }
                return dates;
            },
            getDaysFromNextMonth(numOfDays) {
                let nextMonth = this.nextMonthDates;

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
            selectDay(event) {
                if( !this.selectClick ) {
                    this.selectClick = 1;
                    return this.$set( this.inputDate, 0, event.target.parentElement.getAttribute("data-date") );
                }
                this.selectClick = 0;
                return this.$set( this.inputDate, 1, event.target.parentElement.getAttribute("data-date") );
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
            currentMonthDates() {
                let numberOfDays = this.getNumberOfDaysInMonth(this.date);
                let month = this.date.getMonth();

                let dates = [];
                for( let i = 0; i < numberOfDays; i++ ) {
                    let day = i + 1;

                    let date = new Date( this.today.getFullYear(), month, day );
                    dates.push( this.formattedDate( date ) );
                }
                return dates;
            },
            prevMonthDates() {
                let date = new Date( this.date );
                let prevMonth = new Date( date.setMonth( date.getMonth() - 1 ) );

                let numberOfDays = this.getNumberOfDaysInMonth( prevMonth );
                let month = prevMonth.getMonth();

                let dates = [];
                for( let i = 0; i < numberOfDays; i++ ) {
                    let day = i + 1;

                    let date = new Date( prevMonth.getFullYear(), month, day );
                    dates.push( this.formattedDate( date ) );
                }

                return dates;
            },
            nextMonthDates() {
                let date = new Date( this.date );
                let nextMonth = new Date( date.setMonth( date.getMonth() + 1 ) );

                let numberOfDays = this.getNumberOfDaysInMonth( nextMonth );
                let month = nextMonth.getMonth();

                let dates = [];
                for( let i = 0; i < numberOfDays; i++ ) {
                    let day = i + 1;

                    let date = new Date( nextMonth.getFullYear(), month, day );
                    dates.push( this.formattedDate( date ) );
                }

                return dates;
            },
            currentCalendar() {
                const fields = 42;

                let numberOfDaysInMonth = this.getNumberOfDaysInMonth( this.date );
                let firstDayInMonth = this.getFirstDayOfMonth( this.date );
                let currentMonthDays = this.currentMonthDates;

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

                let dateElements = Array.from( this.$el.querySelectorAll( ".day" ) );
                dateElements.forEach( element => element.classList.remove("day__highlight", "day__highlight--first", "day__highlight--last") );

                let firstDateElement = dateElements.filter( day => day.getAttribute( "data-date" ) === this.inputDate[0] )[0];
                let secondDateElement = dateElements.filter( day => day.getAttribute( "data-date" ) === this.inputDate[1] )[0];

                let highlightDays = dateElements.slice( dateElements.indexOf(firstDateElement), dateElements.indexOf(secondDateElement) + 1 );
                highlightDays.forEach( (element, index) => {
                    if( index === 0 ) {
                        highlightDays[index].classList.add( "day__highlight--first" );
                    }

                    if( index === highlightDays.length - 1 ) {
                        highlightDays[index].classList.add( "day__highlight--last" );
                    }
                    element.classList.add( "day__highlight" );
                } );
            }
        }
    };
</script>

<style lang="scss">
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
        padding: 16px;
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
                color: rgba(0, 0, 0, 0.15);
                width: calc(100% / 7);
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
                    background-color: #00B2F2;
                    border-radius: 100%;
                    color: #FFFFFF;
                    font-weight: bold;
                }

                &__this-month {
                    color: #333333;
                }

                &__highlight {
                    background-color: #CCF1FF;
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