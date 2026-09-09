# Maintainer: Your Name <your@email>
#
# Build confirmed against Folk's actual top-level Makefile:
# - Core binary "folk" links workqueue.o/db.o/trie.o/sysmon.o/epoch.o/
#   folk.o/output-redirection.o/block-stats.o + vendored c11-queues +
#   a statically-built vendor/jimtcl/libjim.a (Jim Tcl is vendored,
#   NOT a system dependency) against -lm -lssl -lcrypto -lz.
# - `make deps` builds vendor/jimtcl in-tree first.
# - Some builtin-programs (e.g. builtin-programs/gpu/draw.folk) JIT-
#   compile additional C at runtime via an embedded `C` object; that
#   needs a working `cc` present on the target system at RUNTIME too,
#   not just at package-build time (gcc is therefore a runtime dep
#   here, not just makedepends).
# - .folk programs are loaded from disk by relative path at runtime,
#   so the package ships the whole source tree, not just the binary.

pkgname=folk-computer-git
pkgver=r1
pkgrel=1
pkgdesc="Physical computing system: reactive database, programming environment, projection mapping"
arch=('x86_64' 'aarch64')
url="https://github.com/FolkComputer/folk"
license=('Apache-2.0')
install=folk.install

depends=(
    'rsync'
    'libjpeg-turbo'
    'libpng'
    'libdrm'
    'pkgconf'
    'v4l-utils'
    'vulkan-tools'
    'vulkan-icd-loader'
    'mesa'
    'shaderc'
    'vulkan-validation-layers'
    'ghostscript'
    'kbd'
    'psmisc'
    'zlib'
    'openssl'
    'systemd'
    'gcc'   # runtime dep: builtin-programs/gpu/draw.folk JIT-compiles
            # C via an embedded `C` object at runtime, not just at
            # package build time.
)

makedepends=(
    'git'
    'cmake'
    'meson'
    'automake'
    'libtool'
    'autoconf-archive'
    'make'
)

# One (or more) of these is required at runtime for a working Vulkan
# ICD, depending on the target GPU. None is force-installed here
# since this is hardware-dependent; the user must pick the right one.
optdepends=(
    'vulkan-intel: Vulkan support for Intel GPUs (ANV driver)'
    'vulkan-radeon: Vulkan support for AMD GCN+ GPUs (RADV driver) -- NOTE: does not cover pre-GCN/TeraScale AMD parts'
    'vulkan-nouveau: Vulkan support for NVIDIA GPUs via open-source driver'
    'seatd: non-root DRM/KMS session access without a full display manager'
)

# Arch has no direct equivalent of Debian's console-data package
# (keymap policy selection). Folk's direct-console keyboard handling
# (kbd_mode/dumpkeys) needs a REAL virtual console device, not a pty
# -- see post_install note below.

source=("git+https://github.com/FolkComputer/folk.git"
        "folk.service")
sha256sums=('SKIP'
            'SKIP')

pkgver() {
    cd folk
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

build() {
    cd folk
    # Build vendored Jim Tcl first (in-tree, statically linked --
    # NOT a system Tcl dependency), then the core `folk` binary.
    # Note: pacman's default CFLAGS may collide with how Folk's
    # Makefile appends its own CFLAGS (e.g. -DTRACY_ENABLE gating the
    # C++/Tracy path) -- unset here to build a plain, predictable
    # release binary. Remove this if you specifically want a Tracy-
    # instrumented build.
    unset CFLAGS
    make deps
    make
}

package() {
    # Install a canonical, read-only reference copy under /usr/share
    # rather than directly under /home -- pacman/namcap generally
    # frown on package-managed files living under /home. The actual
    # runtime copy that Folk reads/writes .folk files from gets synced
    # into /home/folk/folk by post_install()/post_upgrade() below.
    install -d "$pkgdir/usr/share/folk"
    cp -r folk/* "$pkgdir/usr/share/folk/"

    # systemd unit -- includes the TTYPath/PAMName fixes worked out
    # for direct-console keyboard (kbd_mode/dumpkeys) + KMS/Vulkan
    # direct-display access. Calls the runtime binary directly rather
    # than `make start` since that target's systemd-vs-interactive
    # self-detection is a developer-convenience shim, not needed once
    # packaged.
    install -Dm644 folk.service "$pkgdir/usr/lib/systemd/system/folk.service"
}
