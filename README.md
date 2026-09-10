# HTTP Analysis Using Wireshark: Embedded Image Traffic


**Author:** Omolara Animashawun
**Environment:** Kali Linux
**Primary Tool:** Wireshark
**Capture Interface:** Loopback (`lo`)
**Protocol:** HTTP over TCP
**Server:** Apache HTTP Server 2.4.68
**Client:** Web Browser / curl 8.21.0
**Analysis Date:** 10 September 2026

## 🔬 Lab Overview

This practical demonstrates the capture and forensic analysis of plaintext HTTP traffic generated when a web browser retrieves an HTML page containing an embedded JPEG image.

The exercise focuses on how a browser retrieves multiple web resources over HTTP and how those resources appear at the packet, TCP, and application layers.

The laboratory environment consists entirely of locally generated traffic between:

```text
127.0.0.1 → 127.0.0.1
```

The Apache web server hosted the test webpage and image locally. A browser was then used to retrieve the webpage, allowing the resulting HTTP traffic to be captured with Wireshark/TShark.

The captured traffic was subsequently examined to:

* identify the HTTP request for the HTML document;
* identify the separate HTTP request for the embedded JPEG image;
* examine the HTTP response containing the image;
* analyse TCP segmentation and reassembly;
* reconstruct and export the transferred JPEG object;
* verify the recovered object against the original source image using SHA-256;
* compare browser-generated traffic with a curl-only request; and
* explain the difference between browser resource loading and basic command-line HTTP retrieval.

The laboratory therefore demonstrates an end-to-end evidence chain:

```text
Original Source Image
        ↓
HTML Page with Embedded Image
        ↓
Apache HTTP Server
        ↓
Browser HTTP Request
        ↓
Separate Image HTTP Request
        ↓
TCP Segmentation
        ↓
TCP Reassembly
        ↓
HTTP Object Reconstruction
        ↓
Recovered JPEG
        ↓
SHA-256 Verification
        ↓
Browser vs curl Comparison
```

## 🎯 Objectives

The objectives of this practical were to:

1. Create a controlled HTTP webpage containing an embedded image.
2. Host the webpage and image using Apache HTTP Server.
3. Verify that both resources were available through plaintext HTTP.
4. Generate browser traffic by accessing the webpage.
5. Capture the resulting HTTP/TCP traffic using TShark.
6. Analyse the capture using Wireshark.
7. Identify the separate HTTP GET requests generated for the webpage and embedded image.
8. Examine the HTTP response associated with the JPEG image.
9. Demonstrate TCP segmentation and reassembly.
10. Export the reconstructed JPEG object from the packet capture.
11. Verify the recovered image's file type, dimensions, size and SHA-256 digest.
12. Compare the recovered object's SHA-256 digest against the original source image.
13. Perform a separate curl-only request and compare its behaviour with the browser.
14. Preserve the resulting PCAP files and recovered evidence for further examination.

## 🖥️ Laboratory Environment

The practical was performed in an isolated Kali Linux virtual machine.

### Software and tools

| Tool Purpose               |                                            |
| ------------------------- | ------------------------------------------ |
| Kali Linux                | Laboratory operating environment           |
| Apache HTTP Server 2.4.68 | Local HTTP server                          |
| Wireshark 4.6.6           | Graphical packet analysis                  |
| TShark 4.6.6              | Command-line packet capture                |
| curl 8.21.0               | Command-line HTTP client                   |
| ImageMagick               | Image verification and metadata inspection |
| `file`                    | File-type identification                   |
| `sha256sum`               | Cryptographic hash verification            |
| Git                       | Evidence/version-control management        |

### Network configuration

The traffic was intentionally generated over the local loopback interface:

```text
Interface: lo
Source: 127.0.0.1
Destination: 127.0.0.1
HTTP Port: 80
```

Using the loopback interface ensured that the practical traffic remained within the controlled laboratory environment.

## 🖼️ Source Image Verification

A JPEG image was selected as the embedded web object and placed in the laboratory source directory:

```text
working/source/lab_photo.jpg
```

Before the image was introduced into the HTTP environment, its characteristics were verified.

The source image was identified as:

```text
JPEG image data
399 × 501 pixels
8-bit sRGB
35,289 bytes
```

A SHA-256 digest was also calculated before the image was transferred through HTTP.

### Original SHA-256

```text
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce
```

This digest serves as the baseline value for the later forensic comparison.

The purpose of calculating the baseline before network transfer was to establish a trusted reference against which the object recovered from the packet capture could be compared.

## 🌐 Creation of the HTTP Test Objects

Two web objects were prepared:

```text
working/web/
├── image.html
└── lab_photo.jpg
```

The HTML document was deliberately created with an embedded image reference:

```html
<img src="/lab_photo.jpg" alt="Flower vase" width="399" height="501">
```

The relevant HTML structure was:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SBT-DF203 Lab 2 - Embedded Image Traffic</title>
</head>
<body>
    <h1>SBT-DF203 Lab 2</h1>
    <h2>HTTP Analysis Using Wireshark</h2>
    <p>This page contains an embedded image transferred over plaintext HTTP.</p>

    <img src="/lab_photo.jpg" alt="Flower vase" width="399" height="501">
</body>
</html>
```

The embedded image reference is significant because the browser is expected to retrieve the HTML document first and then issue a separate request for the JPEG resource referenced by the `<img>` element.

This provides a controlled way to demonstrate that a webpage can result in multiple HTTP object requests.

## ⚙️ Apache Web Server Configuration

### 🔍 Apache Service Verification

Apache HTTP Server was checked before deployment of the laboratory files.

The service was confirmed to be active and running.

The Apache document root was also inspected before the Lab 2 files were introduced. This established the initial state of the local web server.

The existing document root contained other files and directories, but these were not required for the primary Lab 2 analysis.

### 📂 Deployment of Laboratory Objects

The prepared HTML document and JPEG image were copied into the Apache document root:

```text
/var/www/html/
```

The deployed resources were:

```text
/var/www/html/image.html
/var/www/html/lab_photo.jpg
```

The resulting file sizes were:

```text
image.html      386 bytes
lab_photo.jpg   35 KB
```

The deployment established the two resources that would subsequently be requested by the browser.

### 🌐 HTTP Availability Verification

Before packet capture, both resources were tested using curl.

The HTML resource returned:

```text
HTTP/1.1 200 OK
Content-Length: 386
Content-Type: text/html
```

The JPEG resource returned:

```text
HTTP/1.1 200 OK
Content-Length: 35289
Content-Type: image/jpeg
```

These responses confirmed that both objects were successfully served by Apache over plaintext HTTP.

The values also established useful reference points for the later Wireshark analysis and recovered-object verification.

## 🌍 Browser-Based HTTP Traffic Generation

The test webpage was accessed through a web browser using:

```text
http://127.0.0.1/image.html
```

The webpage rendered successfully and displayed:

* the SBT-DF203 Lab 2 heading;
* the HTTP analysis description; and
* the embedded flower-vase JPEG image.

Successful rendering confirmed that the HTML document and its referenced image were available to the browser.

This step was important because the practical required browser-generated traffic rather than relying exclusively on command-line HTTP requests.

When the browser processed the HTML document, it generated a separate request for the embedded JPEG resource.

## 📡 Browser HTTP Packet Capture

The browser traffic was captured on the local loopback interface using TShark:

```bash
sudo tshark -i lo -f "tcp port 80" -w /tmp/browser_http_capture.pcapng
```

The capture was subsequently moved into the laboratory evidence directory:

```text
evidence/browser_http_capture.pcapng
```

The completed capture contained:

```text
78 packets
```

The traffic was limited to the local HTTP service and therefore showed communication between:

```text
127.0.0.1
```

The capture was then opened in Wireshark for detailed analysis.

## 🔎 Initial Wireshark Analysis

The browser capture was examined in Wireshark.

The packet list demonstrated the establishment of a TCP connection from an ephemeral client port to the Apache HTTP service on port 80.

The capture included HTTP requests and responses associated with:

```text
/image.html
/lab_photo.jpg
/favicon.ico
```

The `/favicon.ico` requests were incidental browser-generated requests and were not part of the primary laboratory object analysis.

The important observation was that the browser generated separate requests for the HTML document and embedded JPEG image.

## 🌐 Identification of HTTP Requests

The Wireshark display filter:

```text
http.request
```

was applied to isolate HTTP request packets.

The resulting request traffic included:

```text
GET /image.html HTTP/1.1
GET /lab_photo.jpg HTTP/1.1
GET /favicon.ico HTTP/1.1
```

The presence of both:

```text
GET /image.html HTTP/1.1
```

and:

```text
GET /lab_photo.jpg HTTP/1.1
```

provides direct packet-level evidence that the webpage and embedded image were requested as separate HTTP resources.

The browser request for the JPEG included the request URI:

```text
/lab_photo.jpg
```

and the host:

```text
127.0.0.1
```

The request also contained browser-generated HTTP headers including a `User-Agent` and `Connection: keep-alive`.

The favicon requests were excluded from the primary analysis because they were not required for the embedded-image traffic demonstration.

## 📨 HTTP Response Analysis

The JPEG response was examined at the HTTP layer.

The relevant packet was identified as:

```text
HTTP/1.1 200 OK
```

and Wireshark identified it as a JPEG/JFIF image response.

The response contained the following significant values:

```text
Status Code: 200 OK
Server: Apache/2.4.68 (Debian)
Content-Length: 35289
```

The response also contained metadata including:

```text
Date
Last-Modified
ETag
Accept-Ranges
```

The `Content-Length` value of 35,289 bytes corresponds to the size of the original JPEG object established before deployment.

This provided an additional consistency check between the source object and the HTTP response.

## 🔗 TCP Segmentation and Reassembly

The JPEG response was analysed at the TCP layer.

Wireshark reported:

```text
[2 Reassembled TCP Segments (35577 bytes): #9(32768), #11(2809)]
```

This indicates that the relevant higher-layer data was distributed across two TCP segments before being reassembled by Wireshark.

The segment sizes were:

```text
Segment 1: 32,768 bytes
Segment 2: 2,809 bytes
-----------------------
Total:     35,577 bytes
```

The reassembled TCP data size is larger than the JPEG's HTTP `Content-Length` of 35,289 bytes.

This difference is expected because the reassembled TCP data associated with the HTTP response includes HTTP response headers in addition to the JPEG entity body.

The observation demonstrates the distinction between:

* TCP segment payloads;
* reassembled TCP application data; and
* the HTTP entity body represented by the JPEG `Content-Length`.

Wireshark's reassembly information therefore provides direct evidence that the HTTP response crossed TCP segment boundaries and was reconstructed at a higher protocol layer.

## 📦 HTTP Object Export

Wireshark's **Export Objects → HTTP** functionality was used to examine application-layer objects reconstructed from the browser capture.

The HTTP Object Export interface identified the transferred image as:

```text
Content Type: image/jpeg
Filename: lab_photo.jpg
Size: approximately 35 kB
```

Multiple occurrences of the JPEG were visible in the object list because the browser generated more than one request for the webpage/image during the captured activity.

The object associated with the relevant HTTP response was selected for recovery.

The successful appearance of `lab_photo.jpg` in the HTTP object list demonstrates that Wireshark was able to reconstruct the JPEG from the captured HTTP/TCP traffic.

## 💾 Recovery of the JPEG Object

The reconstructed image was exported from Wireshark and saved as:

```text
exported/lab_photo_recovered.jpg
```

The recovered file was then independently examined outside Wireshark.

The `file` utility identified it as a valid JPEG:

```text
JPEG image data
399x501
components 3
```

ImageMagick independently reported:

```text
JPEG
399x501
8-bit sRGB
35289B
```

The recovered file therefore retained the same key characteristics as the original source object.

## 🔐 SHA-256 Verification

The recovered JPEG was hashed using SHA-256.

The recovered object produced:

```text
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce
```

The original source image produced the same digest:

```text
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce
```

### Hash comparison

```text
Original:
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce

Recovered:
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce
```

The identical SHA-256 digests demonstrate that the recovered JPEG is byte-for-byte identical to the original source image.

This is stronger than comparing only the filename, dimensions or visual appearance. A matching SHA-256 digest confirms that the complete file contents are identical.

## 🆚 Browser versus curl Behaviour

A separate curl-only capture was performed to compare command-line HTTP retrieval with browser behaviour.

The curl request was:

```bash
curl -v http://127.0.0.1/image.html -o /dev/null
```

The verbose output showed:

```text
> GET /image.html HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/8.21.0
> Accept: */*
```

Apache returned:

```text
< HTTP/1.1 200 OK
< Content-Length: 386
< Content-Type: text/html
```

The separate curl packet capture contained:

```text
GET /image.html HTTP/1.1
HTTP/1.1 200 OK
```

There was no corresponding:

```text
GET /lab_photo.jpg
```

in the curl-only capture.

This demonstrates an important difference between the two clients.

A browser processes the returned HTML and identifies referenced resources such as images, stylesheets and scripts. It can therefore generate additional HTTP requests for those resources.

A basic curl request, by contrast, retrieves the URL explicitly supplied to it. It does not automatically behave like a graphical browser by parsing the returned HTML and requesting every embedded resource.

Therefore:

```text
Browser:
GET /image.html
        ↓
Process HTML
        ↓
GET /lab_photo.jpg
```

Whereas the curl request demonstrated:

```text
curl:
GET /image.html
        ↓
HTTP 200 response
```

If the JPEG were required through curl, it would need to be requested separately.

## 🗃️ Cache and Reload Considerations

Browser caching can affect whether a subsequent page load produces network requests for previously retrieved resources.

If a browser has a valid cached copy of an object, it may be able to use that cached copy rather than downloading the object again. Conversely, a reload, hard refresh, cache-control behaviour, or a request containing cache-related directives can cause the browser to contact the server again.

The browser capture contained repeated requests for the laboratory resources, and some browser requests included:

```text
Pragma: no-cache
```

This is consistent with cache-revalidation or reload-related behaviour and helps explain why multiple occurrences of the same resources appeared in the capture.

The repeated requests should not be interpreted as multiple different image files. They represent repeated HTTP retrieval activity for the same laboratory resource during the captured browser activity.

The curl comparison further demonstrates why client behaviour matters during packet analysis: two clients requesting the same webpage can generate different HTTP request patterns depending on how they process the returned content.

## 🧾 Evidence Integrity

The laboratory maintained a clear relationship between the original source object, captured traffic, reconstructed object and cryptographic verification.

The evidence chain is:

```text
working/source/lab_photo.jpg
        │
        │ SHA-256 baseline
        ↓
Apache HTTP Server
        │
        │ plaintext HTTP
        ↓
evidence/browser_http_capture.pcapng
        │
        │ Wireshark HTTP/TCP analysis
        ↓
HTTP Object Export
        │
        ↓
exported/lab_photo_recovered.jpg
        │
        │ SHA-256 comparison
        ↓
Matching SHA-256 digest
```

The original and recovered images produced the same SHA-256 digest:

```text
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce
```

This establishes the integrity of the recovered image relative to the known source object.

## 📁 Repository Structure

The laboratory materials are organised as follows:

```text
SBT-DF203-Lab2/
│
├── evidence/
│   ├── browser_http_capture.pcapng
│   └── curl_http_capture.pcapng
│
├── exported/
│   └── lab_photo_recovered.jpg
│
├── reports/
│   └── [final report]
│
├── screenshots/
│   └── [lab evidence screenshots]
│
├── scripts/
│   └── [supporting scripts, if applicable]
│
└── working/
    ├── source/
    │   └── lab_photo.jpg
    │
    └── web/
        ├── image.html
        └── lab_photo.jpg
```

### Directory purposes

**`evidence/`**

Contains the packet captures generated during the laboratory:

* browser-generated HTTP traffic;
* curl-only HTTP traffic.

**`exported/`**

Contains the JPEG object recovered from the browser packet capture using Wireshark HTTP Object Export.

**`working/source/`**

Contains the original source image used to establish the SHA-256 baseline.

**`working/web/`**

Contains the webpage and image used to create the controlled HTTP environment.

**`screenshots/`**

Reserved for screenshots documenting the practical evidence and Wireshark analysis.

**`reports/`**

Reserved for the completed practical report and associated documentation.

**`scripts/`**

Reserved for scripts used to support the practical where applicable.

## 🔬 Key Findings

The practical established the following findings:

### 🔹 Finding 1 - Multiple HTTP Objects

The browser did not retrieve the webpage as a single object. Wireshark identified separate HTTP requests for:

```text
GET /image.html HTTP/1.1
GET /lab_photo.jpg HTTP/1.1
```

This demonstrates browser-driven retrieval of an embedded web resource.

### 🔹 Finding 2 - Successful JPEG Retrieval

The server returned:

```text
HTTP/1.1 200 OK
```

for the JPEG resource, with:

```text
Content-Length: 35289
```

and the object was identified as a JPEG/JFIF image.

### 🔹 Finding 3 - TCP Segmentation

Wireshark reported:

```text
2 Reassembled TCP Segments
```

with segment contributions of:

```text
32768 bytes
2809 bytes
```

showing that the HTTP response crossed TCP segment boundaries and was subsequently reassembled.

### 🔹 Finding 4 - Successful Object Reconstruction

Wireshark successfully reconstructed and exported:

```text
lab_photo.jpg
```

as a valid JPEG object.

### 🔹 Finding 5 - Cryptographic Integrity

The original and recovered images produced the identical SHA-256 digest:

```text
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce
```

This demonstrates byte-for-byte identity between the source and recovered objects.

### 🔹 Finding 6 - Browser and curl Behave Differently

The browser generated a request for the embedded image after retrieving the HTML page.

The curl-only capture showed only:

```text
GET /image.html
```

and its corresponding HTTP response.

This demonstrates that browser resource loading and basic curl retrieval produce different HTTP traffic patterns.

## 📊 Evidence Summary

The practical evidence is documented through screenshots covering the complete analysis process.

The evidence sequence includes:

| Evidence            |                     Description                         |
| ------------------- | ------------------------------------------------------- |
| Figure 3            | Original source image verification and baseline SHA-256 |
| Figure 4            | HTML page and embedded image preparation                |
| Figure 5            | Initial Apache service and document-root verification   |
| Figure 6            | Deployment of Lab 2 web objects                         |
| Figure 7            | HTTP availability verification                          |
| Figure 8            | Browser rendering of the laboratory webpage             |
| Figure 10           | Verification of browser HTTP packet capture             |
| Figure 11           | Initial Wireshark analysis                              |
| Figure 12           | Identification of browser HTTP requests                 |
| Figure 13           | HTTP response for embedded JPEG                         |
| Figure 14           | TCP segmentation and reassembly                         |
| Figure 15           | Wireshark HTTP Object Export                            |
| Figure 16           | Verification of recovered JPEG                          |
| Figure 17           | SHA-256 comparison                                      |
| Figure 18           | Verbose curl request                                    |
| Figure 19           | Packet-level curl comparison                            |

The evidence sequence provides a continuous technical narrative from source preparation through network capture, protocol analysis, object recovery and cryptographic verification.

## 📝 Conclusion

This practical demonstrated how plaintext HTTP traffic can be analysed from the network-packet level through to the recovery and verification of an application-layer object.

A controlled Apache web server hosted an HTML document containing an embedded JPEG image. When the page was accessed through a browser, Wireshark captured separate HTTP requests for the HTML document and the embedded image.

The JPEG response was then examined at the TCP and HTTP layers. Wireshark identified two TCP segments contributing to the reassembled response, demonstrating TCP segmentation and reassembly.

The reconstructed JPEG was exported from the packet capture and independently verified. The recovered image retained the original 399 × 501 dimensions and 35,289-byte size. Most importantly, its SHA-256 digest exactly matched the digest calculated from the original source image:

```text
25f4073c345856b582500b9a6f59b38248984081b2dac90e25a54270113ad8ce
```

This confirms that the recovered image is byte-for-byte identical to the original source object.

A separate curl-only capture further demonstrated the difference between browser and command-line HTTP behaviour. The browser requested both the HTML page and its embedded image, while the basic curl request retrieved only the explicitly requested HTML resource.

Overall, the exercise demonstrates a complete workflow for analysing plaintext HTTP traffic, identifying individual web objects, understanding TCP segmentation and reassembly, reconstructing transferred content, and validating recovered evidence using cryptographic hashing.

## ⚖️ Ethical and Laboratory Scope

All traffic analysed in this practical was generated within an isolated, controlled laboratory environment using the local loopback interface.

The analysis was limited to intentionally created laboratory resources hosted on the local Apache server.

No third-party, production, or unauthorised network traffic was intentionally captured or analysed as part of this exercise.

The techniques demonstrated here are intended for authorised cybersecurity training, digital forensics, network analysis and controlled laboratory environments.
