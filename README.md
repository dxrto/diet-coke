# Diet Coke

DevOps hate him!! Lose 5 layers with this simple trick!!!

Read the blog post I wrote about this project: [Medium.com](https://medium.com/@eater./diet-coke-14e329f740e3)

---

```
diet-coke - one weird trick DevOps engineers don't want you to know!

    diet-coke [-i <file>] [-s] [-o <file>] [-m <comment>] [-g <image>] <layer>-[<layer>]
    diet-coke [-i <file>] -l

Options:
    -i  Input file, when - is given STDIN is used, default: -
    -o  Output file, when - is given STDOUT is used, default: -
    -m  Comment to put on the new compressed layer, on default a summary of the old layers will be used
    -g  On which image diet coke should work
    -s  Suffer, even in STDOUT is a tty, write to STDOUT
    -l  List layers in given save file
    -h  Show this help
```

# Requirements

- Perl5
- the `JSON::XS` module (`perl-JSON-XS` in Void, `libjson-xs-perl` in Ubuntu (maybe?), worst case `cpan JSON::XS`)

## Example

I have the following image with the tag `d.xr.to`

```
docker image save d.xr.to/php-fpm | ./diet-coke -l
INFO: Selected image: d.xr.to/php-fpm:latest
 0 sha256:ecd79b3c89d02ddb8ee196f7d3d2f774ff15165201907efff607cf8d1f0c590c EXT Comment: d.xr.to/base:toybox initialization from chroot
                                                                           LABEL maintainer==@eater.me
 1 sha256:427dcf68ddc4c7cedbecd9b7b5dc6155c41f3162a7c707642f3e3b5cc10307ee RUN xbps-install -Sy php php-mysql php-sqlite wget
 2 sha256:85b8f036d44d34bc01d564126f33fbdabf0ae98b65fb8618f7fe9f17d3a33ce4 RUN sed magic
 3 sha256:bdc0724f8772c14445badc9c10e6ce3170996ebb9b06bcd382a7a218f8d1daec COPY file:0f661c9788bdae933722b863e02d65dc0d563f6c05dae05dca91ccd2ba954b1e in /bin/composer-install.sh
 4 sha256:fe1729db674a2b8ea513652488592835d84091b2ecbf2dea1b8b4fa6d2685eb6 RUN bash /bin/composer-install.sh
 5 sha256:f41e7bd5797ad858ad794497310d606b91896e0b6306c66c4df1facdd63b248c RUN rm -f /bin/composer-install.sh
 6 sha256:d51c776b6fbb4e9c804f9a89166d8e28faa72613266251e8e981c870a51ec477 RUN xbps-install -Sy php-fpm
 7 sha256:eea88523a513e77460a488694511f4ac985952574ed78a44117956b9e053dc6a RUN useradd -rU www -u 444
                                                                           RUN sed 's:\(user\|group\)\s*=\s*http:\1 = www:'
                                                                           EXPOSE 9000/tcp
                                                                           CMD ["/usr/bin/php-fpm" "--nodaemonize"]
```

I now want to compress everything from layer 1 into 1 layer. this can be done by running `./diet-coke 1-`

```
[eater@momo diet-coke]$ docker image save d.xr.to/php-fpm | ./diet-coke -s 1- | ./diet-coke -l
INFO: Selected image: d.xr.to/php-fpm:latest
INFO: Compressing layers 1 to and including 7
INFO: overwrite: etc/php/php.ini
INFO: new: usr/bin/composer-install.sh
INFO: new: .composer/
INFO: clean: .composer/
INFO: new: .composer/keys.dev.pub
INFO: new: .composer/keys.tags.pub
INFO: new: usr/bin/composer
INFO: remove: usr/bin/composer-install.sh
INFO: new: etc/php/php-fpm.conf
INFO: new: etc/php/php-fpm.d/
INFO: clean: etc/php/php-fpm.d/
INFO: new: etc/php/php-fpm.d/www.conf
INFO: new: etc/sv/
INFO: clean: etc/sv/
INFO: new: etc/sv/php-fpm/
INFO: new: etc/sv/php-fpm/run
INFO: new: etc/sv/php-fpm/supervise
INFO: new: usr/bin/php-fpm
INFO: new: usr/share/man/man8/
INFO: new: usr/share/man/man8/php-fpm.8
INFO: new: usr/share/php/
INFO: clean: usr/share/php/
INFO: new: usr/share/php/fpm/
INFO: new: usr/share/php/fpm/status.html
INFO: new: var/cache/xbps/php-fpm-7.2.12_1.x86_64-musl.xbps
INFO: new: var/cache/xbps/php-fpm-7.2.12_1.x86_64-musl.xbps.sig
INFO: new: var/db/xbps/.php-fpm-files.plist
INFO: overwrite: var/db/xbps/pkgdb-0.38.plist
INFO: new: etc/group
INFO: new: etc/group-
INFO: new: etc/passwd
INFO: new: etc/passwd-
INFO: new: etc/shadow
INFO: new: etc/shadow-
INFO: Done
INFO: Selected image: d.xr.to/php-fpm:latest
 0 sha256:e3f702b8760ca8c7a1bece4b0f3d07aba13b1b4855698263e11827f5626f6246 EXT Comment: d.xr.to/base:toybox initialization from chroot
 1 sha256:0cddcda9a1429eefd18fb6956d977933d799e8b00f8dd0fa7ab80de13ab7a656 EXT Comment: diet coke: d.xr.to/php-fpm:latest
                                                                           RUN sed 's:\(user\|group\)\s*=\s*http:\1 = www:'
                                                                           EXPOSE 9000/tcp
                                                                           CMD ["/usr/bin/php-fpm" "--nodaemonize"]

```

It now compressed all layers from 1 successfully into a new layer, and we can also just import this straight into docker again, with the following command:

```
docker image save d.xr.to/php-fpm | ./diet-coke -s 1- | docker image load
```

Since the image is tagged, your `d.xr.to/php-fpm` image will now be replaced with a compressed version.