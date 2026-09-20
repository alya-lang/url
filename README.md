# url

[![CI](https://github.com/alya-lang/url/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/url/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/url?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Furl%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Furl%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

WHATWG and RFC 3986 compliant URL parser, serializer, normalizer, reference resolver, and query string library for Alya.

---

## 🌟 Features

- 🌐 **Standards Compliant**: Full implementation of RFC 3986 and WHATWG URL principles (hierarchical and opaque schemes).
- 🔍 **Comprehensive Parsing**: Extracts scheme, username, password, host (IPv4, IPv6 bracketed `[::1]`, domains), port, path, query, and fragment.
- ⚡ **High Performance**: Parses ~250,000 URLs/sec and formats ~2,000,000 URLs/sec with minimal allocations.
- 🔀 **Reference Resolution (`join`)**: RFC 3986 Section 5 base reference resolution supporting absolute URLs, protocol-relative, absolute path, query-only, fragment-only, and directory traversal (`../`).
- 🧹 **RFC 3986 Normalization**: Lowercases scheme and host, removes standard default ports (80 for http, 443 for https), eliminates dot segments (`.`, `..`), and normalizes percent-encodings.
- 📋 **WHATWG `UrlSearchParams`**: Full query parameter manipulation container supporting duplicate keys, `get`, `get_all`, `has`, `set`, `append`, `delete`, `sort`, `to_string`, and `to_map`.
- 🛠️ **Fluent `UrlBuilder`**: Clean builder API for constructing and mutating complex URLs incrementally.

---

## 📁 Project Architecture

```
url/
├── alya.toml               # Package manifest
├── src/
│   ├── lib.alya            # Public API facade & high-level functions
│   ├── types.alya          # Url, UrlSearchParams, and UrlBuilder structs
│   └── core/
│       ├── percent.alya    # RFC 3986 percent encoding and decoding
│       ├── path.alya       # Path segments, dot-segment removal & reference join
│       ├── parser.alya     # Parsing, formatting, origin & default port catalog
│       ├── query.alya      # Query parsing, format & WHATWG UrlSearchParams
│       ├── builder.alya    # Fluent URL builder implementation
│       └── normalize.alya  # RFC 3986 canonical URL normalization
├── examples/
│   └── demo.alya           # Comprehensive runnable showcase
├── tests/
│   ├── test_parser.alya    # URL parser, origin & port catalog test suite
│   ├── test_percent.alya   # Percent encoding/decoding test suite
│   ├── test_path.alya      # Dot segment removal & reference resolution tests
│   ├── test_query.alya     # Query strings & UrlSearchParams tests
│   ├── test_builder.alya   # Fluent builder tests
│   └── test_normalize.alya # Canonical normalization tests
└── benches/
    └── bench_basic.alya    # Micro-benchmarks
```

---

## 📦 Installation

Add `url` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
url = { git = "https://github.com/alya-lang/url", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alya add url --git https://github.com/alya-lang/url --branch main
alya install
```

---

## 🚀 Quick Start

### 1. Parsing and Inspecting URLs

```alya
import "url"

function main()
    let u = url::parse("https://alice:secret@api.example.com:8443/v1/users?role=admin#top")
    say u.scheme     # "https"
    say u.username   # "alice"
    say u.host       # "api.example.com"
    say u.port       # "8443"
    say u.path       # "/v1/users"
    say u.query      # "role=admin"
    say u.fragment   # "top"

    say url::url_origin(u)          # "https://api.example.com:8443"
    say url::url_effective_port(u)  # "8443"
    say url::url_is_https(u)        # 1
end

main()
```

### 2. Query Parameters (`UrlSearchParams`)

```alya
import "url"

function main()
    let sp = url::search_params_from("category=books&tag=tech&tag=alya&sort=price")
    say url::search_params_get(sp, "category")    # "books"
    
    let tags = url::search_params_get_all(sp, "tag")
    say tags[0] # "tech"
    say tags[1] # "alya"

    url::search_params_set(sp, "sort", "rating")
    url::search_params_append(sp, "page", "1")
    url::search_params_sort(sp)

    say url::search_params_to_string(sp)
    # "category=books&page=1&sort=rating&tag=tech&tag=alya"
end

main()
```

### 3. Fluent URL Builder

```alya
import "url"

function main()
    let b = url::builder()
    url::builder_scheme(b, "https")
    url::builder_host(b, "api.github.com")
    url::builder_path(b, "/repos")
    url::builder_append_path(b, "alya-lang/url")
    url::builder_param(b, "per_page", "100")
    url::builder_fragment(b, "readme")

    let target_url = url::builder_to_string(b)
    say target_url
    # "https://api.github.com/repos/alya-lang/url?per_page=100#readme"
end

main()
```

### 4. Reference Resolution (`join`)

```alya
import "url"

function main()
    let base = "https://example.com/api/v1/catalog"

    say url::join(base, "items")          # "https://example.com/api/v1/items"
    say url::join(base, "/v2/items")       # "https://example.com/v2/items"
    say url::join(base, "?page=2")         # "https://example.com/api/v1/catalog?page=2"
    say url::join(base, "../health")       # "https://example.com/api/health"
    say url::join(base, "//cdn.test/img")  # "https://cdn.test/img"
end

main()
```

### 5. URL Normalization

```alya
import "url"

function main()
    let raw = "HTTP://EXAMPLE.COM:80/a/b/../c/./d?z=2&a=1"
    say url::normalize(raw)
    # "http://example.com/a/c/d?a=1&z=2"
end

main()
```

---

## 📖 API Reference

### Core URL API

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `parse(raw_url)` | `string` | `Url` | Parses raw URL into a `Url` struct. |
| `format(u)` / `to_string(u)` | `Url` | `string` | Serializes `Url` struct into a URL string. |
| `url_origin(u)` | `Url` | `string` | Returns the origin string (`scheme://host[:port]`). |
| `url_is_https(u)` | `Url` | `int` | Returns `1` if scheme is `https`, `0` otherwise. |
| `url_is_http(u)` | `Url` | `int` | Returns `1` if scheme is `http`, `0` otherwise. |
| `url_default_port(scheme)` | `string` | `string` | Returns the default port for scheme (e.g. `"80"`, `"443"`). |
| `url_effective_port(u)` | `Url` | `string` | Returns explicit port or default port for the scheme. |
| `url_path_segments(u)` | `Url` | `array` | Returns list of non-empty path segments. |
| `url_remove_dot_segments(path)` | `string` | `string` | RFC 3986 5.2.4 dot segment removal (`.`, `..`). |
| `join(base, relative)` | `string, string` | `string` | Resolves a relative reference against base URL. |
| `normalize(raw_url)` | `string` | `string` | Canonical normalization (lowercasing, default port removal, query sort). |

### Percent Encoding API

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `url_encode(text)` | `string` | `string` | RFC 3986 percent encoding for unreserved characters. |
| `url_decode(text)` | `string` | `string` | RFC 3986 percent decoding (handles `+` as space). |
| `url_encode_component(text)` | `string` | `string` | Percent-encodes component string. |
| `url_decode_component(text)` | `string` | `string` | Percent-decodes component (preserves literal `+`). |
| `url_encode_path(text)` | `string` | `string` | Percent-encodes path while preserving `/`. |

### `UrlSearchParams` API

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `search_params()` | none | `UrlSearchParams` | Creates empty parameter container. |
| `search_params_from(query)` | `string` | `UrlSearchParams` | Parses query string into parameter container. |
| `search_params_from_map(m)` | `map` | `UrlSearchParams` | Creates container from Map `{ key: val }`. |
| `search_params_get(sp, key)` | `sp, string` | `string` | Returns first value for key or `""`. |
| `search_params_get_all(sp, key)` | `sp, string` | `array` | Returns all values matching key. |
| `search_params_has(sp, key)` | `sp, string` | `int` | Returns `1` if key exists, `0` otherwise. |
| `search_params_set(sp, k, v)` | `sp, string, string` | `void` | Replaces all matches with single key-value pair. |
| `search_params_append(sp, k, v)` | `sp, string, string` | `void` | Appends a new key-value pair. |
| `search_params_delete(sp, key)` | `sp, string` | `void` | Deletes all entries with specified key. |
| `search_params_sort(sp)` | `sp` | `void` | Sorts parameters in-place alphabetically by key. |
| `search_params_to_string(sp)` | `sp` | `string` | Formats parameters as percent-encoded query string. |
| `search_params_to_map(sp)` | `sp` | `map` | Converts parameters to Map. |
| `search_params_keys(sp)` | `sp` | `array` | Returns list of unique keys. |
| `search_params_len(sp)` | `sp` | `int` | Returns count of entries. |
| `search_params_clear(sp)` | `sp` | `void` | Clears all entries. |

### Fluent `UrlBuilder` API

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `builder()` | none | `UrlBuilder` | Creates a new fluent builder. |
| `builder_from_url(u)` | `Url` | `UrlBuilder` | Initializes builder from existing `Url`. |
| `builder_from_string(raw_url)` | `string` | `UrlBuilder` | Initializes builder by parsing URL string. |
| `builder_scheme(b, scheme)` | `b, string` | `UrlBuilder` | Sets scheme (e.g. `"https"`). |
| `builder_credentials(b, u, p)` | `b, string, string` | `UrlBuilder` | Sets username and password. |
| `builder_host(b, host)` | `b, string` | `UrlBuilder` | Sets host domain or IP. |
| `builder_port(b, port)` | `b, string` | `UrlBuilder` | Sets explicit port. |
| `builder_path(b, path)` | `b, string` | `UrlBuilder` | Sets path. |
| `builder_append_path(b, seg)` | `b, string` | `UrlBuilder` | Appends a path segment safely. |
| `builder_param(b, key, val)` | `b, string, string` | `UrlBuilder` | Appends a query parameter. |
| `builder_query(b, query_str)` | `b, string` | `UrlBuilder` | Parses and appends query string. |
| `builder_fragment(b, frag)` | `b, string` | `UrlBuilder` | Sets fragment anchor. |
| `builder_build(b)` | `b` | `Url` | Builds `Url` struct. |
| `builder_to_string(b)` | `b` | `string` | Builds and formats to URL string. |

---

## 🧪 Running Tests & Benchmarks

Run the test suite using `alya`:

```bash
alya test
```

Run individual test files:

```bash
alya run tests/test_parser.alya
alya run tests/test_percent.alya
alya run tests/test_path.alya
alya run tests/test_query.alya
alya run tests/test_builder.alya
alya run tests/test_normalize.alya
```

Run benchmarks:

```bash
alya run benches/bench_basic.alya
```

Run the example showcase:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   alya fmt . --check
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
