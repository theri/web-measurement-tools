This repository contains tools to automatically download web pages and analyze page loads, e.g., extracting performance metrics such as Page Load Time.

These tools were tested on Debian 9 (stretch), using Firefox 62.0.2 and har-export-trigger-0.6.1 (the latter is provided with this repository).

To make a packet capture, your local user needs permissions to use tcpdump.
See capture.sh script for instructions for Linux.


How to load pages
=================

`cd load`

With default list of URLs and packet capture (if your current user can use tcpdump):

`./capture.sh`

Without packet capture:

`./run.sh`

With custom list of URLs, more than once per URL, and/or custom scenario log string:

1. Provide a list of URLs in a file, one URL per line
2. Call `./capture.sh ./run.sh $URLFILE $HOW_MANY_TIMES_PER_URL $SCENARIO_LOGSTRING $LOG_DIR`
3. Find data in $LOG_DIR (defaults to ../testdata/run-$(date +%Y-%m-%dT%H:%M)-test)

How to compute data based on page load data
=================

*Note:* This requires the data be available in a directory called "data" within the "compute" directory.
By default, this links to the ../testdata directory.

**Step 2 requires at least one dataset to be present in "data/", containing at least navtiming.log, HAR files, and Resource Timings log files.** Additionally, computetimings.py can check which runs were successful, for which it requires workload_output.log and urlfile-$SCENARIO_LOGSTRING.log within the dataset. If a .pcap file is present as well, it can analyze the "failure modes" for failed page loads.

**Step 3 requires a pcap file to be present within the data set, as generated by capture.sh, see above.**

1. `cd compute`
2. `./computetimings.py`
3. `./validate_object_sizes.py`

Step 2 outputs:
* (If checking for succeeded): Which page loads failed and which succeeded (to terminal and success_or_fail.log)
* (For succeeded, or all): Summary of HAR file and Resource Timings (to terminal and final_timings.log)
* (For succeeded, or all): Comparison of all object sizes and whether they are in HAR, Resource Timings, or both (to compare_har_res.log)

Step 3 outputs:
* Page loads considered (to terminal)
* All objects successfully read from packet capture trace, with matching HAR file and Resource Timings objects, if available (to object_sizes_trace.log)


How to plot and evaluate computed data
=================

*Note:* This requires the data be available in a directory called "data" within the "compute" directory.
By default, this links to the ../testdata directory.

**You need to run the "computetimings.py" script on the dataset first, see the section above.**

`cd eval`

Scripts output the following data to the terminal and the "plots/" subdirectory within dataset:
* impact_on_metrics.R -- Impact of different data sources on load times and Byte Index
* plot_redirects.R -- Plot number of redirects and time taken up by them
* validate_object_sizes.R -- Compare packet trace ground truth to HAR and Resource Timings
* plot_compare_object_sizes.R -- Compare individual object sizes for all objects
* compare_number_of_objects_and_page_size.R -- Compare numbers of objects in page load and total page size

Overview of scripts
===================

Download web pages
-----------

	load/*
        capture.sh                      Set up packet capture, then call run.sh (or other)
        run.sh                          Create log directory, then run fetchurl.sh and log its output
        fetchurl.sh                     Fetch URLs from a file 1 or more times, log data (see above)
        load_url_using_marionette.py    Fetch a URL using Firefox and Marionette
                                        Log Navigation Timings, Resource Timings, and a HAR file
        load_url_using_chrome.py        Fetch a URL using Chrome and DevTools, log data (see above)

        load_url_using_selenium.py      Fetch a URL using Selenium and geckodriver, log data (see above)

Compute metrics from data
-------------------------

    compute/*
        computetimings.py                Calculate Byte Index, redirect times, succeeded or failed page loads...
        hartimings.py                    Log important parts of HAR file contents to a CSV (used by computetimings)
        get_starttimestamp_from_workload_output        Read workload_output and log all pages and starttimestamps to starttimings.log (called by computetimings)
        get_trace_for_timestamps        For failed page loads, read packet capture trace and dump DNS and HTTP
        validate_object_size.py            From packet capture trace, calculate ground truth object sizes and match them to HAR and Res
        filter_out_chrome_overhead.sh    Filter out DNS queries and connections not connected to page load
        filter_out_firefox_overhead.sh    Filter out DNS queries and connections not connected to page load



Evaluate generated data
------------------------

    eval/*
        impact_on_metrics.R            Impact of different data sources on load times and Byte Index
        plot_redirects.R -- Plot number of redirects and time taken up by them
        validate_object_sizes.R        Compare packet trace ground truth to HAR and Resource Timings
        plot_compare_object_sizes.R    Compare individual object sizes for all objects
        compare_number_of_objects_and_page_size.R    Compare numbers of objects in page load and total page size
        plottimings.R                Helper functions to read and plot metrics
        R_functions.R                Generic helper functions

Misc
====

Dataset
-------

You can find a dataset consisting of 50 runs, each loading 1000 Web pages, at http://dx.doi.org/10.14279/depositonce-8100.

This dataset contains navtimings.log, HAR files, and Resource Timings from all runs presented in the "Web Performance Pitfalls" paper, see below.

Acknowledgement
---------------

If you use this software, or derivatives of it, in your research, please:

1. Include the link to this repository
2. Cite the following publication:

Enghardt, T., Zinner, T., Feldmann, A. (2019). Web Performance Pitfalls. In Passive and Active Measurement Conference 2019 (PAM).

License
-------

This software is released under the [BSD 2-clause license](https://github.com/theri/web-measurement-tools/blob/master/LICENSE.md).
