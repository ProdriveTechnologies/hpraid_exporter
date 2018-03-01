# Prometheus exporter for HP RAID controllers

This exporter for [the Prometheus monitoring system](https://prometheus.io/)
calls into the `ssacli` utility to provide metrics for errors reported by HP
RAID hardware. Under the hood, it invokes the following command when being
scraped:

    ssacli ctrl all diag file=/tmp/...

This command generates a ZIP file containing a diagnostic report. The
exporter opens the ZIP file and extracts the `ADUReport.xml` file. It
then reports all of the active error conditions from all of the
`<Errors/>` blocks.

There are at least two other Prometheus metrics exporters for HP RAID
controllers ([1](https://github.com/gdm85/hpraidmon),
[2](https://github.com/chromium58/hpraid_exporter)). Both these
exporters are built around the idea of parsing the utility's
user-readable output using regular expressions, as opposed to relying on
the utility's XML output. This makes them a lot more brittle.

Though this exporter only provide metrics for errors for the time being
(`hpraid_errors`), it should be fairly easy to extend it to expose other
attributes. All of those metrics should already be part of the
`ADUReport.xml` file that is used by this exporter.

## Using this exporter

The following shell script around Docker can be used to build a
statically linked executable that can be deployed on virtually any
x86-based Linux system:

    ./build_static.sh

The following command will start the exporter, causing it to listen on
TCP port 9423:

    ./hpraid_exporter

Do note that it expects `ssacli` to be in `$PATH`. It also needs to
run as a user that is capable of running this command successfully
(`root`).

Example metrics output:

    # HELP hpraid_errors Errors in the diagnostic report reported by the RAID controller.
    # TYPE hpraid_errors gauge
    hpraid_errors{device="ArrayController=Smart Array P410i in Embedded Slot",message="The cache is temporarily disabled.",severity="Warning"} 1
    # HELP hpraid_scrape_success Whether scraping the HP RAID controller stats was successful.
    # TYPE hpraid_scrape_success gauge
    hpraid_scrape_success 1
