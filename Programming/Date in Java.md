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
        Date date = getLocalDateObject("2016-01-03 10:10:00 +0000");
        DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss Z");
        // You can set another time zone 
        //formatter.setTimeZone(TimeZone.getTimeZone("GMT+8"));

        System.out.println(formatter.format(date)); // Be your local format

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
	}

    public static Date getLocalDateObject(String data) {
        Date date = null;
        // formater.getDateInstance can be more efficient
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
            date = formatter.parse(data);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}

```