
1. [Miscellaneous](#misc)
    * [Prefer nanoTime over currentTimeMillis](#misc_currentTimeMillis_vs_nanoTime)
    * [Prefer URI over URL](#misc_uri_url)
    * [Prefer existing well-tested methods over reinventing the wheel](#misc_well_tested_method)

## <a name='misc'>Miscellaneous</a>

### <a name='misc_currentTimeMillis_vs_nanoTime'>Prefer nanoTime over currentTimeMillis</a>

When computing a *duration* or checking for a *timeout*, avoid using `System.currentTimeMillis()`. Use `System.nanoTime()` instead, even if you are not interested in sub-millisecond precision.

`System.currentTimeMillis()` returns current wallclock time and will follow changes to the system clock. Thus, negative wallclock adjustments can cause timeouts to "hang" for a long time (until wallclock time has caught up to its previous value again).  This can happen when ntpd does a "step" after the network has been disconnected for some time. The most canonical example is during system bootup when DHCP takes longer than usual. This can lead to failures that are really hard to understand/reproduce. `System.nanoTime()` is guaranteed to be monotonically increasing irrespective of wallclock changes.

Caveats:
- Never serialize an absolute `nanoTime()` value or pass it to another system. The absolute value is meaningless and system-specific and resets when the system reboots.
- The absolute `nanoTime()` value is not guaranteed to be positive (but `t2 - t1` is guaranteed to yield the right result)
- `nanoTime()` rolls over every 292 years. So if your Spark job is going to take a really long time, you may need something else :)


### <a name='misc_uri_url'>Prefer URI over URL</a>

When storing the URL of a service, you should use the `URI` representation.

The [equality check](http://docs.oracle.com/javase/7/docs/api/java/net/URL.html#equals(java.lang.Object)) of `URL` actually performs a (blocking) network call to resolve the IP address. The `URI` class performs field equality and is a superset of `URL` as to what it can represent.

### <a name='misc_well_tested_method'>Prefer existing well-tested methods over reinventing the wheel</a>

When there is an existing well-tesed method and it doesn't cause any performance issue, prefer to use it. Reimplementing such method may introduce bugs and requires spending time testing it (maybe we don't even remember to test it!).

  ```scala
  val beginNs = System.nanoTime()
  // Do something
  Thread.sleep(1000)
  val elapsedNs = System.nanoTime() - beginNs

  // This is WRONG. It uses magic numbers and is pretty easy to make mistakes
  val elapsedMs = elapsedNs / 1000 / 1000

  // Use the Java TimeUnit API. This is CORRECT
  import java.util.concurrent.TimeUnit
  val elapsedMs2 = TimeUnit.NANOSECONDS.toMillis(elapsedNs)

  // Use the Scala Duration API. This is CORRECT
  import scala.concurrent.duration._
  val elapsedMs3 = elapsedNs.nanos.toMillis
  ```

Exceptions:
- Using an existing well-tesed method requires adding a new dependency. If such method is pretty simple, reimplementing it is better than adding a dependency. But remember to test it.
- The existing method is not optimized for our usage and is too slow. But benchmark it first, avoid premature optimization.