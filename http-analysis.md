# Network Exchange Analysis

This document details three HTTP network exchanges recorded during a session on Archidekt (`https://archidekt.com`).

---

## 1. Request Records

### Request 1: Main HTML Document
* **Request Method:** `GET`
* **Request URL:** `https://archidekt.com/`
* **Response Status Code:** `200 OK`
* **Headers:**
  * `cache-control: private, no-cache, no-store, max-age=0, must-revalidate` - Forces the browser to never store or reuse a cached copy of the homepage without revalidating directly with the server.
  * `content-encoding: gzip` - Indicates that the HTML body was compressed using Gzip before transmission to minimize data transfer size.

### Request 2: Next.js Stylesheet
* **Request Method:** `GET`
* **Request URL:** `https://cdn.archidekt.com/_next/static/css/af4d862225966922.css`
* **Response Status Code:** `200 OK (from disk cache)`
* **Headers:**
  * `cache-control: public,max-age=31536000,immutable` - Tells the browser this file will never change, allowing it to cache the file locally for 31536000 seconds without re-requesting it.
  * `access-control-allow-origin: *` - Implements Cross-Origin Resource Sharing to permit web pages hosted on archidekt.com to safely load assets hosted on the cdn.archidekt.com sub-domain.

### Request 3: Site Logo Asset
* **Request Method:** `GET`
* **Request URL:** `https://archidekt.com/images/archidekt2.svg`
* **Response Status Code:** `304 Not Modified`
* **Headers:**
  * `etag: W/"3fdc-1a0bbfc6720"` - Provides a unique validation hash representing the current version of the SVG file on the server.
  * `cache-control: public, max-age=0` - Allows public caching, but requires the browser to check back with the server before displaying the cached image.

---

## 2. Network Performance & Protocol Analysis

### Slowest Request Analysis
The initial HTML document request was the slowest exchange out of the 3 I selected. Unlike static resources which are served instantly from local storage, fetching the base document requires opening a fresh TCP connection and waiting for backend server processing. In comparison, the CSS file loaded almost instantly because its status code was `200 OK (from disk cache)`, meaning the browser retrieved it entirely from local hardware storage without sending a network packet.

### Header & Status Code Behavior
The HTTP response status codes and headers directly control how the browser handles local memory and network requests. The `200 OK` status for the main HTML file signaled a successful download. For the SVG image, the server responded with `304 Not Modified`. Because the browser sent an `If-None-Match` request containing the previous etag, the server confirmed the logo had not changed and sent an empty response body, instructing the browser to reuse its existing local copy. Furthermore, the `cache-control: immutable` header on the Next.js CSS file told the browser it never needs to revalidate that specific stylesheet during page reloads.

### Key Takeaway / Surprise
One surprising observation was Archidekt's hybrid infrastructure setup shown across the response headers. The site routes static CSS assets through Google Cloud Storage (`server: UploadServer`, `x-goog-generation`) on a dedicated CDN subdomain (`cdn.archidekt.com`), while serving the main HTML and vector assets through a reverse proxy (`via: 1.1 google`) supporting HTTP/3 (`alt-svc: h3=":443"`). Just had me nerding out a bit.