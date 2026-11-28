# devianter

devianter is a Go client for DeviantArt's guest API. It is the library behind
[SkunkyArt](https://github.com/krz-labs/skunky-art).

This is a maintained fork of
[skunky/devianter](https://git.macaw.me/skunky/devianter).

If you find it useful, you can support the project through
[GitHub Sponsors](https://github.com/sponsors/ccleberg).

## Contents

- [Overview](#overview)
- [Installation](#installation)
- [Usage](#usage)
- [Rate limiting](#rate-limiting)
- [Contributing](#contributing)
- [License](#license)

## Overview

devianter talks to the internal JSON API that deviantart.com's own web
frontend uses. It is not DeviantArt's official, documented API: there is no
application registration and no OAuth. The library authenticates the way a
logged-out browser does, so everything it can reach is what an anonymous
visitor can see. Because the endpoints are internal, DeviantArt can change or
remove them without notice.

Supported data:

- Deviations
- Comments
- Daily deviations
- Search, including searching within a gallery
- Users and groups, with their galleries and favourites
- Avatars and emoji

Full API documentation is on
[pkg.go.dev](https://pkg.go.dev/github.com/krz-labs/devianter).

## Installation

```sh
go get github.com/krz-labs/devianter
```

## Usage

Call `UpdateCSRF` once before anything else. It fetches the guest session
cookie and CSRF token that every other call depends on.

```go
if err := devianter.UpdateCSRF(); err != nil {
	log.Fatal(err)
}

post, apiErr := devianter.GetDeviation("123456789", "someuser")
if apiErr.Reason != "" {
	log.Fatal(apiErr.Error)
}
fmt.Println(post.Deviation.Title, post.IMG)
```

Most functions return an `Error` struct rather than a Go `error`. It is never
nil; a call succeeded if `Error.Reason` is empty.

The session does not refresh itself. A long-running program should call
`UpdateCSRF` again when calls start failing.

## Rate limiting

DeviantArt sits behind CloudFront, which blocks IP addresses that make too
many requests. A blocked request returns an `Error` that mentions
CloudFront/WAF. devianter does no rate limiting, retries or backoff of its
own, so callers making bulk requests need to pace themselves. Set `UserAgent`
to identify your client and `Timeout` to bound each request.

## Contributing

The repository, issue tracker and pull requests are on
[github.com/krz-labs/devianter](https://github.com/krz-labs/devianter).

## License

GNU Affero General Public License v3.0. See [LICENSE](LICENSE).
