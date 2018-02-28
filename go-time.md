> date & time
```
package util

import (
	"time"
)

// 日期、时间处理相关

// 某年某周的第一天
func FirstDayOfISOWeek(year int, week int) time.Time {
	date := time.Date(year, 0, 0, 0, 0, 0, 0, time.Now().Local().Location())
	isoYear, isoWeek := date.ISOWeek()

	// iterate back to Monday
	for date.Weekday() != time.Monday {
		date = date.AddDate(0, 0, -1)
		isoYear, isoWeek = date.ISOWeek()
	}

	// iterate forward to the first day of the first week
	for isoYear < year {
		date = date.AddDate(0, 0, 7)
		isoYear, isoWeek = date.ISOWeek()
	}

	// iterate forward to the first day of the given week
	for isoWeek < week {
		date = date.AddDate(0, 0, 7)
		isoYear, isoWeek = date.ISOWeek()
	}

	return date
}

// 某年某周的最后一天
func LastDayOfISOweek(year int, week int) time.Time {
	dateTime := FirstDayOfISOWeek(year, week)
	dateTime = time.Date(dateTime.Year(), dateTime.Month(), dateTime.Day()+6, 23, 59, 59, 0, time.Now().Local().Location())
	return dateTime
}

// 某年某月的第一天
func FirstDayOfMonth(year int, month int) time.Time {
	firstDay := time.Date(year, time.Month(month), 1, 0, 0, 0, 0, time.Now().Local().Location())
	return firstDay
}

// 某年某月的最后一天
func LastDayOfMonth(year int, month int) time.Time {
	firstDay := time.Date(year, time.Month(month), 1, 0, 0, 0, 0, time.Now().Local().Location())
	lastDay := firstDay.AddDate(0, 1, 0).Add(-time.Nanosecond)
	return lastDay
}

// 上个月最后一天
func FirstDayOfLastMonth(year int, month int) time.Time {
	date := time.Date(year, time.Month(month), 1, 0, 0, 0, 0, time.Now().Local().Location())
	if month == 1 {
		date = date.AddDate(-1, 11, 0)
	} else {
		date = date.AddDate(0, -1, 0)
	}
	return date
}

// 某天第一秒和最后一秒
func GetFirstAndLastSecondOfDate(dateTime time.Time) (time.Time, time.Time) {
	firstSecond := time.Date(dateTime.Year(), dateTime.Month(), dateTime.Day(), 0, 0, 0, 0, time.Now().Local().Location())
	lastSecond := time.Date(dateTime.Year(), dateTime.Month(), dateTime.Day(), 23, 59, 59, 0, time.Now().Local().Location())
	return firstSecond, lastSecond
}

// 某天第一秒和最后一秒（字符串）
func GetFirstAndLastSecondOfDateString(dateTime time.Time) (string, string) {
	firstSecond, lastSecond := GetFirstAndLastSecondOfDate(dateTime)
	return firstSecond.Format("2006-01-02 15:04:05"), lastSecond.Format("2006-01-02 15:04:05")
}

// time.RFC3339 to `yyyy-mm-dd hh:mm:ss`
func ParseRFC3339ToYMDHMS(timeStr string) string {
	timeparsed, err := time.Parse(time.RFC3339, timeStr)
	if err != nil {
		return ""
	}
	return timeparsed.Format("2006-01-02 15:04:05")
}

// time.RFC3339 to `yyyy-mm-dd hh:mm`
func ParseRFC3339ToYMDHM(timeStr string) string {
	timeparsed, err := time.Parse(time.RFC3339, timeStr)
	if err != nil {
		return ""
	}
	return timeparsed.Format("2006-01-02 15:04")
}

// time.RFC3339 to `yyyy-mm-dd hh:mm:ss`
func ParseRFC3339ToYMD(timeStr string) string {
	timeparsed, err := time.Parse(time.RFC3339, timeStr)
	if err != nil {
		return ""
	}
	return timeparsed.Format("2006-01-02")
}

// time.RFC3339 to `hh:mm:ss`
func ParseRFC3339ToHMS(timeStr string) string {
	timeparsed, err := time.Parse(time.RFC3339, timeStr)
	if err != nil {
		return ""
	}
	return timeparsed.Format("15:04:05")
}

// time.RFC3339 to `hh:mm`
func ParseRFC3339ToHM(timeStr string) string {
	timeparsed, err := time.Parse(time.RFC3339, timeStr)
	if err != nil {
		return ""
	}
	return timeparsed.Format("15:04")
}

```
