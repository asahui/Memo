#Date, Calendar, DateFormat in Java

###Hints

* Date is not timezone aware
* Many methods are deprecated in Date, use Calendar instead
* Calendar and DateFormat are timezone aware
* Calendar change a Date Object into timezone aware Calendar Object and vice versa
* DateFormat format a Date Object into a String under specific timezone vice versa


###Example Code



```java
import java.util.*;
import java.lang.*;
import java.io.*;
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.TimeZone;

class DateExample
{
	public static void main (String[] args) throws java.lang.Exception
	{
        Date date = getLocalDateObjectFromStringWithoutTimeZone("2016-01-03 10:10:00 +0800"); //even timezone is provided, assume as GMT
        DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss Z");
        // You can set another time zone 
        //formatter.setTimeZone(TimeZone.getTimeZone("GMT+8"));

        System.out.println(formatter.format(date)); // Be your local format
        System.out.println(getLocalDateString(date)); // same

        // Calendar use default timezone
        Calendar c = Calendar.getInstance();

        // date (no timezone info) -> calendar (with timezone info)
        // if you do not know which timezone is `date` in, it will be inappropriate
        // Since date is parse by DateFormat as a Local Date, it is right to use
        // default Canlendar
        c.setTime(date);
		System.out.println(c.get(Calendar.HOUR_OF_DAY)); // Be your local hour

        // Change timezone
        c.setTimeZone(TimeZone.getTimeZone("GMT"));
        System.out.println(c.get(Calendar.HOUR_OF_DAY)); // Should be 10
        System.out.println(getTimeZoneDateString(date, TimeZone.getTimeZone("GMT"))); // same

        // Calendar usage
        Calendar c1 = Calendar.getInstance();
        c1.set(2016, 1, 3, 10, 10);  // 2016-02-03 10:10:00 +1100   month starts from 0
        System.out.println(formatter.format(c1.getTime())); // Be your local format
        System.out.println(getLocalDateString(c1.getTime())); // same
	}

    /**
     * Format local date to String in info(RFC) format
     */
    public static String getLocalDateString(Date date) {
        TimeZone tz = TimeZone.getDefault();
        return getTimeZoneDateString(date, tz);
    }

    /**
     * @link http://www.mkyong.com/java/java-convert-date-and-time-between-timezone/
     */
    public static String getTimeZoneDateString(Date date, final TimeZone tz) {
        DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss Z");
        formatter.setTimeZone(tz);
        return formatter.format(date);
    }

    /**
     * @param date A date string in info(RFC) format yyyy-MM-dd HH:mm:ss Z
     * Z is the timezone information that must be included
     *
     * @return A Date Object
     */
    public static Date getLocalDateObject(String date) {
        TimeZone tz = TimeZone.getDefault();
        return getTimeZoneDateObject(date, tz);

    }

    /**
     * get a date from a string(with timezone information) and convert to another Timezone @param tz
     *
     * @param date A date string in info(RFC) format yyyy-MM-dd HH:mm:ss Z
     * Z is the timezone information that must be included
     *
     * @param tz converted timezone
     *
     * @return A Date Object
     */
    public static Date getTimeZoneDateObject(String date, TimeZone tz) {
        Date d = null;

        DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss Z");
        formatter.setTimeZone(tz);

        try {
            d = formatter.parse(date);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return d;
    }


    /**
     * get Local Date Object from a string(without timezone information)
     * assume the timezone of the date string is GMT.
     *
     * @param date A date string in info(RFC) format yyyy-MM-dd HH:mm:ss
     *
     * @return A Date Object represent local date
     */
    public static Date getLocalDateObjectFromStringWithoutTimeZone(String date) {
        Date d = null;
        // formater.getDateInstance can be more efficient

        // method1: parse with timezone and using default timezone
        // DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss Z");
        // //formatter.setTimeZone(TimeZone.getDefault());

        // method 2: parse without timezone
        DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        formatter.setTimeZone(TimeZone.getTimeZone("GMT"));


        try {
            // Just parse it and get the date object, which is not timezone aware.
            // 1. If you want to handle timezone information, use Calendar or DateFormat.
            //
            // 2. Calendar could parse a date as a Calendar Object and you can use this 
            // Calendar object to get all time, date, timezone information.
            //
            // 3. DateFormat could format a date associate with a timezone you set, but 
            // throw away the timezone information when parsing a date pattern. 
            //
            // DateFormat use the default timezone from your computer if you do not 
            // implicitly set it.
            // 
            // You can parse a date pattern consist of timezone info(RFC) like this
            //   DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss Z");
            // Or you can set a timezone and parse a date pattern without time zone
            //   DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            //   formatter.setTimeZone(TimeZone.getTimeZone("GMT"));
            //
            d = formatter.parse(date);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return d;
    }
}
```
