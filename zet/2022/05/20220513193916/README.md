# NixOS go app using cgo and proxyVendor

When changin from `proxyVendor=false` to `proxyVendor=true` in a 
`buildGoModule` in NixOS it is essential to also change the `vendorSha256`
otherwise the vendor folder form `go mod vendor` is used instead of the
one from `go mod download`. This leads to the problem that c header files
from cgo dependencies are not fetched and thus not available. It is as easy
as setting a fake `vendorSha256` and then getting the right one from the
error during build.

