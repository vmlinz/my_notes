# Location Hal Implementation #

Android location HAL utilizes the android HAL framework to implement a hardware and kernel independent module. When porting or implementing GPS feature on Android platform, you just need to implement several standard GPS HAL interfaces defined by Android. So that when android framework initializes, it will new a thread for location manager service, and load gps HAL shared library and call the interface functions you provided at the proper time.

With Android location HAL, we don't have to care about how the framework works, so we just focus on implementing the interfaces defined by location HAL, then the location manager service will work as expected.

## HAL ##

Please refer to [HAL implementation](hal.markdown)

## Location Manager Service ##

**Android location architecture**

![location architecture](https://github.com/vmlinz/my_notes/raw/master/location_arch.jpeg)

**Android location manager service initializition**

![location manager service](https://github.com/vmlinz/my_notes/raw/master/location_init.jpeg)

## Location HAL Interfaces ##

* `com_android_server_location_GpsLocationProvider.cpp` implements the interfaces between location manager service and HAL module.

* `{"native_init", "()Z", (void*)android_location_GpsLocationProvider_init}` exports the HAL init function to java framework. when this function is called, hal module will be load into memory and return GpsInterface struct to jni with gps callbacks pointing to the real implementations in hal.

* `{"native_start", "()Z", (void*)android_location_GpsLocationProvider_start}` android provider will call this native function which is implemented in location hal which will start gps hardware for location.

* after android java framework call into hal to start location, then the location thread will call the callback functions provided to report location changes to android java framework, so the application have a chance to know the location changes and so on.

## Driver Interfaces ##

So according to sample gps implementation for us, the kernel part should provide us at least these interfaces:

### gps serial device interface ###

* **device node:** `/dev/ttyS0 or something else`
* **standard file operations like** `open, close, read, write`
* **read function** which give us standard nmea data line by line
* **write function** which give us a standard interface to control gps hardware

### gps io control ###

* `gps_state_start`
* `gps_state_stop`

### gps transport control ###

* `gps_dev_set_nmea_message_rate`
* `gps_dev_set_baud_rate`

## Data Structures ##

* struct GpsLocation;

	Typedef struct {
	/** set to sizeof(GpsLocation) */
	size_t          size;
	/** Contains GpsLocationFlags bits. */
	uint16_t        flags;
	/** Represents latitude in degrees. */
	double          latitude;
	/** Represents longitude in degrees. */
	double          longitude;
	/** Represents altitude in meters above the WGS 84 reference
	* ellipsoid. */
	double          altitude;
	/** Represents speed in meters per second. */
	float           speed;
	/** Represents heading in degrees. */
	float           bearing;
	/** Represents expected accuracy in meters. */
	float           accuracy;
	/** Timestamp for the location fix. */
	GpsUtcTime      timestamp;
	} GpsLocation;

<table>
<tbody>
<tr>
<th>type</th>
<th>value</th>
<th>description<>
</tr>
<tr>
<td>size_t</td>
<td>size</td>
<td>size of struct GPSLocation</td>
</tr>
<tr>
<td>uint16_t</td>
<td>flags</td>
<td>Contains GpsLocationFlags bits</td>
</tr>
<tr>
<td>double</td>
<td>latitude</td>
<td>Represents latitude in degrees</td>
</tr>
<tr>
<td>double</td>
<td>longitude</td>
<td>Represents longitude in degrees</td>
</tr>
<tr>
<td>double</td>
<td>altitude</td>
<td>Represents altitude in meters above the WGS 84 reference ellipsoid</td>
</tr>
<tr>
<td>float</td>
<td>speed</td>
<td>Represents speed in meters per second
</td>
</tr>
<tr>
<td>float</td>
<td>bearing</td>
<td>Represents heading in degrees</td>
</tr>
<tr>
<td>float</td>
<td>accuracy</td>
<td>Represents expected accuracy in meters</td>
</tr>
<tr>
<td>GpsUtcTime</td>
<td>timestamp</td>
<td>Timestamp for the location fix
</tr>
</tbody>
</table>

* struct GpsStatus;

	/** Represents the status. */
	typedef struct {
		/** set to sizeof(GpsStatus) */
		size_t          size;
		GpsStatusValue status;
	} GpsStatus;

<table>
<tbody>
<tr>
<th>type</th>
<th>value</th>
<th>description<>
</tr>
<tr>
<td>size_t</td>
<td>size</td>
<td>set to sizeof(GpsStatus)</td>
</tr>
<tr>
<td>GpsStatusValue</td>
<td>status</td>
<td></td>
</tr>
</tbody>
</table>

* struct GpsSvInfo;

	/** Represents SV information. */
	typedef struct {
	/** set to sizeof(GpsSvInfo) */
	size_t          size;
	/** Pseudo-random number for the SV. */
	int     prn;
	/** Signal to noise ratio. */
	float   snr;
	/** Elevation of SV in degrees. */
	float   elevation;
	/** Azimuth of SV in degrees. */
	float   azimuth;
	} GpsSvInfo;

<table>
<tbody>
<tr>
<th>type</th>
<th>value</th>
<th>description<>
</tr>
<tr>
<td>int</td>
<td>prn</td>
<td>Pseudo-random number for the SV.</td>
</tr>
<tr>
<td>float</td>
<td>snr</td>
<td>Signal to noise ratio.</td>
</tr>
<tr>
<td>float</td>
<td>elevation</td>
<td>Elevation of SV in degrees.</td>
</tr>
<tr>
<td>float</td>
<td>azimuth</td>
<td>Azimuth of SV in degrees.</td>
</tr>
</tbody>
</table>

## Files ##

* location provider jni `(frameworks/base/services/jni/com_android_server_location_GpsLocationProvider.cpp)`
* location provider `(frameworks/base/services/java/com/android/server/location/GpsLocationProvider.java)`
* location manager service `(frameworks/base/services/java/com/android/server/LocationManagerService.java)`
* [freerunner_gps.c](http://git.android-x86.org/?p=platform/hardware/gps.git;a=blob;f=gps.c;h=199de46e7708262b37a61ad1706e7cde93ebccd7;hb=c044569632a80c01f032c8726e783e3728c2d5cc)

## Resources ##

* [Android GPS Analysis](http://hi.baidu.com/%CB%EF%CC%EF%BB%AA/blog/item/60ff6e2964bc4921359bf732.html)
* [Android GPS HAL](http://blog.chinaunix.net/space.php?uid=20485710&do=blog&id=1666975)
* [Android HAL Introduction](http://www.slideshare.net/jollen/android-hal-introduction-libhardware-and-its-legacy)
