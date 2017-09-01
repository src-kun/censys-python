Censys Python Library
=====================

This is a light weight Python wrapper to the Censys REST API.

Install
-------

The egg can be installed using Pip or easy_install (e.g., `sudo pip install censys`).

Usage
=====

There are three API endpoints that the library provides access to: Index,
Query, and Export. More details about each can be found in the Censys API
documentation: https://censys.io/api.


Search/View/Report API
----------------------

The index APIs allow you to perform full-text searches, view specific records,
and generate aggregate reports about the IPv4, Websites, and Certificates
endpoints. There is a Python class for each index: `CensysIPv4`,
`CensysWebsites`, and `CensysCertificates`. Below, we show an example for
certificates, but the same methods exist for each of the three indices.

#CensysCertificates
```python
import censys.certificates

c = censys.certificates.CensysCertificates(api_id="XXX", api_secret="XXX")

# view specific certificate
print c.view("a762bf68f167f6fbdf2ab00fdefeb8b96f91335ad6b483b482dfd42c179be076")

# iterate over certificates that match a search
fields = ["parsed.subject_dn", "parsed.fingerprint_sha256"]
for cert in c.search("github.com and valid_nss: true", fields=fields):
	print cert["parsed.subject_dn"]

# aggregate report on key types used by trusted certificates
print c.report(query="valid_nss: true", field="parsed.subject_key_info.key_algorithm.name")

```

#CensysIPv4
```
import censys.ipv4

UID = ""
SECRET = ""

ipv4 = censys.ipv4.CensysIPv4(UID, SECRET)
fields = ["parsed.subject_dn", "parsed.fingerprint_sha256", "parsed.fingerprint_sha1"]

for c in ipv4.search("80.http.get.headers.server: nginx"):
	print c
```
#CensysWebsites

```
import censys.websites

UID = ""
SECRET = ""

CensysWebsites = censys.websites.CensysWebsites(UID, SECRET)
fields = ["parsed.subject_dn", "parsed.fingerprint_sha256", "parsed.fingerprint_sha1"]

for c in CensysWebsites.search("80.http.get.headers.server: nginx"):
	print c
```

SQL Query API
-------------

The SQL Query endpoint allows running SQL against the indices.

```python
import censys.query

c = censys.query.CensysQuery(api_id="XXX", api_secret="XXX")

# find datasets (e.g., ipv4) that have exposed data
print c.get_series()

# get schema and tables for a given dataset
print c.get_series_details("ipv4")

# Start SQL job
res = c.new_job("select count(*) from certificates.certificates")
job_id = res["job_id"]

# Wait for job to finish and get job metadata
print c.check_job_loop(job_id)

# Iterate over the results from that job
print c.get_results(job_id, page=1)

```


SQL Export API
--------------

The SQL Export API allows exporting large subsets of data using SQL.

```python
import censys.export

c = censys.export.CensysExport(api_id="XXX", api_secret="XXX")

# Start new Job
res = c.new_job("select count(*) from certificates.certificates")
job_id = res["job_id"]

# Wait for job to finish and fetch results
print c.check_job_loop(job_id)

```

Data API
--------

The Data API allows programatic access to the raw data files.

```python
import censys.data

c = censys.data.CensysData(api_id="XXX", api_secret="XXX")

# Get a Series
ssh_series = c.view_series('22-ssh-banner-full_ipv4')

# View all the files in each scan
for scan in ssh_series['results']['historical']:
    print c.view_result('22-ssh-banner-full_ipv4', scan['id'])
```
