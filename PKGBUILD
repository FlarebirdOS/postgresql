pkgname=(
    postgresql
    postgresql-libs
    postgresql-docs
)
pkgbase=postgresql
pkgver=18.beta1.r832.g879c492480d
pkgrel=2
pkgdesc="Sophisticated object-relational DBMS"
arch=('x86_64')
url="https://www.postgresql.org"
license=('PostgreSQL')
depends=(
    'bash'
    'gcc-libs'
    'glibc'
    'icu'
    'krb5'
    'libldap'
    'libxml2'
    'libxslt'
    'linux-pam'
    'llvm-libs'
    'lz4'
    'openssl'
    'readline'
    'systemd-libs'
    'util-linux-libs'
    'zlib'
    'zstd'
)
makedepends=(
    'clang'
    'docbook-xml'
    'docbook-xsl'
    'git'
    'llvm'
    'perl'
    'perl-ipc-run'
    'python'
    'systemd'
    'tcl'
    'util-linux'
)
source=(git+https://git.postgresql.org/git/postgresql.git
    0001-Set-DEFAULT_PGSOCKET_DIR-to-run-postgresql.patch
    0002-Force-RPATH-to-be-used-for-the-PL-Perl-plugin.patch
    postgresql-check-db-dir.in
    postgresql.logrotate
    postgresql.pam
    postgresql.service)
sha256sums=(SKIP
    2c09429dca9caf540be647fdac9540eeccb68935994bb54cfd3f2108464916c7
    0fb4915c06b9767933b27adc329e7319485e043fb9f17b1697b969779a00cf14
    94af93b53bf7772e6664c239523ef952ffc905a0de3c2c4b2dfc2fe8f3a2efed
    6abb842764bbed74ea4a269d24f1e73d1c0b1d8ecd6e2e6fb5fb10590298605e
    8b54acedcffbff1421b05d8383ebbee67d9e2a3aedec9a0e1fc6ed7635a0797b
    3a0bbf9fd61259c4898465c55a8b7cd26202885132ff2b5c685dfe0e0ddd4968)

pkgver() {
    cd postgresql

    git describe --tags | sed 's/REL_//;s/_/./;s/-/.r/;s/-/./g' | tr A-Z a-z
}

prepare() {
    cd postgresql

    patch -p1 < ${srcdir}/0001-Set-DEFAULT_PGSOCKET_DIR-to-run-postgresql.patch
    patch -p1 < ${srcdir}/0002-Force-RPATH-to-be-used-for-the-PL-Perl-plugin.patch
}

build() {
    cd postgresql

    local configure_args=(
        --sysconfdir=/etc
        --mandir=/usr/share/man
        --datadir=/usr/share/postgresql
        --disable-rpath
        --enable-nls
        --enable-tap-tests
        --with-gssapi
        --with-icu
        --with-ldap
        --with-libxml
        --with-libxslt
        --with-llvm
        --with-lz4
        --with-openssl
        --with-pam
        --with-perl
        --with-python
        --with-readline
        --with-system-tzdata=/usr/share/zoneinfo
        --with-systemd
        --with-tcl
        --with-uuid=e2fs
        --with-zstd
        ${configure_options}
    )

    # Fix static libs
    CFLAGS+=" -ffat-lto-objects"

    ./configure "${configure_args[@]}"

    make world
}

package_postgresql() {
    pkgdesc="Sophisticated object-relational DBMS"
    depends+=("postgresql-libs>=${pkgver}")
    backup=(
        etc/logrotate.d/postgresql
        etc/pam.d/postgresql
    )
    options+=('staticlibs')
    install=postgresql.install

    cd postgresql

    # install
    make DESTDIR=${pkgdir} install
    make -C contrib DESTDIR=${pkgdir} install
    make -C doc/src/sgml DESTDIR=${pkgdir} install-man

    # we don't want these, they are in the -libs package
    for dir in src/interfaces src/bin/pg_config src/bin/pg_dump src/bin/psql src/bin/scripts; do
        make -C ${dir} DESTDIR=${pkgdir} uninstall
    done

    for util in pg_config pg_dump pg_dumpall pg_restore psql \
        clusterdb createdb createuser dropdb dropuser pg_isready reindexdb vacuumdb; do
        rm ${pkgdir}/usr/share/man/man1/${util}.1
    done

    # clean up unneeded installed items
    rm -rf ${pkgdir}/usr/include/postgresql/internal
    rm -rf ${pkgdir}/usr/include/libpq
    find ${pkgdir}/usr/include -maxdepth 1 -type f -execdir rm {} +
    rmdir ${pkgdir}/usr/share/doc/postgresql/html


    install -Dm 755 ${srcdir}/postgresql-check-db-dir.in ${pkgdir}/usr/bin/postgresql-check-db-dir

    install -Dm 644 ${srcdir}/${pkgname}.pam ${pkgdir}/etc/pam.d/${pkgname}
    install -Dm 644 ${srcdir}/${pkgname}.logrotate ${pkgdir}/etc/logrotate.d/${pkgname}

    install -Dm 644 ${srcdir}/${pkgname}.service -t ${pkgdir}/usr/lib/systemd/system

    install -d -o 41 -g 41 -m 700 ${pkgdir}/var/lib/postgres
    install -d -o 41 -g 41 -m 700 ${pkgdir}/var/lib/postgres/data
}

package_postgresql-libs() {
    pkgdesc="Libraries for use with PostgreSQL"
    depends=(
        'glibc'
        'krb5'
        'libldap'
        'lz4'
        'openssl'
        'readline'
        'zlib'
        'zstd'
    )

    cd postgresql

    # install libs and non-server binaries
    for dir in src/interfaces src/bin/pg_config src/bin/pg_dump src/bin/psql src/bin/scripts; do
        make -C ${dir} DESTDIR=${pkgdir} install
    done

    for util in pg_config pg_dump pg_dumpall pg_restore psql \
        clusterdb createdb createuser dropdb dropuser pg_isready reindexdb vacuumdb; do
        install -Dm 644 doc/src/sgml/man1/${util}.1 ${pkgdir}/usr/share/man/man1/${util}.1
    done

    install -d ${pkgdir}/usr/include/{libpq,postgresql/internal/libpq}

    # these headers are needed by the public headers of the interfaces
    install -m 644 src/include/pg_config.h ${pkgdir}/usr/include
    install -m 644 src/include/pg_config_manual.h ${pkgdir}/usr/include
    install -m 644 src/include/pg_config_os.h ${pkgdir}/usr/include
    install -m 644 src/include/postgres_ext.h ${pkgdir}/usr/include
    install -m 644 src/include/libpq/libpq-fs.h ${pkgdir}/usr/include/libpq

    # these he aders are needed by the not-so-public headers of the interfaces
    install -m 644 src/include/c.h ${pkgdir}/usr/include/postgresql/internal
    install -m 644 src/include/port.h ${pkgdir}/usr/include/postgresql/internal
    install -m 644 src/include/postgres_fe.h ${pkgdir}/usr/include/postgresql/internal
    install -m 644 src/include/libpq/pqcomm.h ${pkgdir}/usr/include/postgresql/internal/libpq
}

package_postgresql-docs() {
    pkgdesc="HTML documentation for PostgreSQL"
    depends=()
    options+=('docs')

    cd postgresql

    make -C doc/src/sgml DESTDIR=${pkgdir} install-html

    chown -R root:root ${pkgdir}/usr/share/doc/postgresql/html

    # clean up
    rmdir ${pkgdir}/usr/share/man/man{1,3,7}
    rmdir ${pkgdir}/usr/share/man
}
