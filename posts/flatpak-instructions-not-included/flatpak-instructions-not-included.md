---
title: "Flatpak: Instructions Not Included"
date: 2021-05-22T19:08:56+01:00
draft: false
featuredImage: static/postimages/flatpak-instructions-not-included/flatpak.png
author: Luke Briggs
rssFullText: true
linkToMarkdown: true
categories: []
description: How German Laser Beams Fixed My App
toc:
  enable: true
---

There is a very peculiar thing I have found in software development, it is incredibly difficult to actually get software on other people's machines. Even compiled languages can have difficulty.

## Context
I am elbow deep in a very large personal project at the moment (all will be revealed soon) that has so far taken a month. The project is getting to a stage now where deployment methods have to be evaluated. From the beginning I wanted the project to be fully cross platform across Windows, Linux, Mac with a singular codebase. Because of these specific requirements, I went with the [Qt GUI framework](https://www.qt.io/product/framework), it is cross platform, runs natively on each platform, and has bindings for Python (my preferred language).

Because nothing is ever easy, there are actually two different python bindings for Qt. There is [PySide](https://en.wikipedia.org/wiki/PySide) which was created by the Qt Company themselves. It has a less restrictive license, but seems to have less of a community around it due to it being the newer of the two bindings. The other binding is called [PyQt](https://en.wikipedia.org/wiki/PyQt) and came first, even being called `python3-qt` in the fedora package manager. When it comes down to coding, there is pretty much no difference between them since they are all just pythonic fronts for the same C++ back-end. For the whole project up until recently I was using PySide since it is the same in every way except for a better license.

## Python Deployment
Python, as it is shipped as CPython, does not (by default) compile to machine code. You'll notice a parenthetical and a subordinate clause in the previous sentence, that's how windy this road goes.

There are systems that will turn your python code into binary executables, I was using [PyInstaller](https://www.pyinstaller.org/) to get the job done for me. I'd run it on Linux, Windows, MacOS and all my code would be packaged in to binary form; there was the issue of non-python dependencies I had but I could find ways around that. So there I was with all my binaries in a row and feeling confident about how I'm going to deploy my application.

## Linux Binary Woes
My current main machine runs Fedora Linux, this is a distribution that is kept reasonably up to date and, at the time of writing, has a new desktop environment and newer kernel than other non-Arch distros. All of this meant that when I tried to run my Linux compiled program on Ubuntu (A major distro I want to support), I get thrown an error that Ubuntu doesn't have the correct version of glibc (the dynamically linked C libraries). Sticking with the standard glibc versions that come with each distro, GNU/Linux is backwards compatible but not forwards compatible. That is to say that a program compiled on an older distro will run on a newer distro but not vice-versa. The solution to this seems straightforward doesn't it? Compile the program on an older version. So I loaded up an Ubuntu LTS VM, compile the program, works like a charm. I try to run that same executable on my main Fedora machine and I get an error message:

```cmd
Settings schema 'org.gnome.settings-daemon.plugins.xsettings'
does not contain a key named 'antialiasing'
```

The program does not run. This is without doubt a bug in Gnome 40. But it is a bug that happens 100% of the time, it may get fixed, it may not. My application is either incompatible with anything other than my own machine, or compatible with everything *except* my own machine. Of course I could have two separate binaries, but that would be a bit of a hassle to maintain. In comes what I thought would be my saviour, Flatpak.

### Build Systems
Linux has 3 main methods of deploying applications. Shipping binaries, making per distribution packages, and containers (of which flatpak is one). Binaries are, as we've seen, version dependent, packages are distribution dependent, and containers run across all machines but have extra bulk since they package there own dependencies rather than relying on system libraries.

Throughout my exploration of deployment options, I have made all three. I have documented my results of binaries, and I also went on to make an RPM package (the packaging system that Fedora uses).

```sh
Name:           Pepys
Version:        0.3.0
Release:        8%{?dist}
Summary:        A straightforward markdown journal

License:        GPLv3
URL:            https://www.lukebriggs.dev/pepys
Source0:        pepys.tar.gz

Requires:       python3, python3-wheel, python3-pip, pandoc, enchant, wkhtmltopdf, python3-qt5, python3-qt5-webengine, python3-regex, python3-num2words, python3-pypandoc, python3-enchant, python3-setproctitle, texlive-mdwtools
BuildRequires:  python3-pip
%description

%build
MAIN_DIR=%{buildroot}/usr/share/pepys
APP_DIR=%{buildroot}/usr/local/share/applications
mkdir -p ${MAIN_DIR}/temp
mkdir -p ${APP_DIR}

tar -xzvf %{SOURCE0} -C ${MAIN_DIR}/temp
if test -f ${MAIN_DIR}/temp/src/resources/base/config.json; then
rm ${MAIN_DIR}/temp/src/resources/base/config.json
fi
if test -f ${MAIN_DIR}/temp/src/resources/base/wordlist.txt; then
rm ${MAIN_DIR}/temp/src/resources/base/wordlist.txt;
fi

cp -r ${MAIN_DIR}/temp/* ${MAIN_DIR}
rm -rf ${MAIN_DIR}/temp


printf "[Desktop Entry]\n \
Type=Application\n \
Name=Pepys\n \
Categories=Office;\n \
X-GNOME-FullName=Pepys\n \
Comment=A straightforward markdown journal\n \
Icon=%s/src/main/resources/base/icons/appicons/icon.svg\n \
NoDisplay=false\n \
Exec= python3 %s/src/main/python/main.py\n \
Path=\n \
Terminal=false\n \
X-GNOME-UsesNotifications=false\n \
StartupWMClass=Pepys" "/usr/share/pepys" "/usr/share/pepys" > $APP_DIR/Pepys.desktop
pip3 install PyPDF4


%files
/*

%post
chmod -R 777 /usr/share/pepys

#%license add-license-file-here
#%doc add-docs-here



%changelog
* Sat May 15 2021 Luke Briggs <lukebriggs02@gmail.com>
- 
```

**NOTICE: The above is a script used for testing, it is mostly functional but don't use it as a guide for how to do things properly**

The above is a script you feed into `rpmbuild` and it will generate a .rpm file which Fedora users can use to install your software. You can see that all it really is is some metadata listing app information, dependencies (these dependencies are ones that are inside the fedora packaging system), and a shell script to say what gets installed and how. Because all the dependencies are listed, and are all from the the distros package manager, the file size of an rpm only consists of the files that you supply. In my case it was some python source files and some vector icons, totalling 1.2MB. Of course the full size of the app still needs to be downloaded but the other 150MB or so come in the form of system libraries that other apps can also use. This is the classic Linux method of software deployment, it is lean, and ensures compatibility with a particular distro. The problem comes when looking at other distributions. Some distributions also use RPM (CentOS, RHEL, OpenSUSE) but have different packages, and so need special cases in the file. Other distributions use a different package management system, Debian and derivatives use .deb files for instance.

### Flatpak
[Flatpak](https://en.wikipedia.org/wiki/Flatpak) and it's rival [Snap](https://en.wikipedia.org/wiki/Snap_(package_manager)) aim to solve the dependency hell by shipping everything together in one big package. You avoid incompatibility at the cost of redundancy. The cover image of this post is my flatpak spec file that gets fed into `flatpak-builder`. And you may notice that it works in a similar way to a distribution's own package manager.

We have metadata along with base dependencies
```yaml
app-id: dev.lukebriggs.pepys
runtime: org.kde.Platform
runtime-version: '5.15'
sdk: org.kde.Sdk
add-extensions:
  org.freedesktop.Sdk.Extension.texlive:
    directory: texlive
    subdirectories: true
    autodelete: true
    version: '20.08'
command: runner.sh
```

Because everything is in a sandbox, we have to declare what parts of the host system we have to use
```yaml
finish-args:
  - --share=ipc
  - --socket=x11
  - --socket=wayland
  - --socket=pulseaudio
  - --device=dri
  - --filesystem=home
  - --share=network
  - --env=PATH=/usr/bin:/app/bin:/app/texlive/bin:/app/texlive/bin/x86_64-linux:/app/texlive/bin/aarch64-linux
```

And then comes the rest of our dependencies which have to be files we supply, along with the build script we use to actually do what we want with the files.

#### The Trouble with Python, Qt and Flatpak
All was going swimmingly until I had to get my Qt app in flatpak.
2 of my app dependencies are installed by effectively copying the prebuild binaries into the flatpak sandbox.
```yaml
  - name: pandoc
    buildsystem: simple
    build-commands:
      - ls
      - install -D bin/pandoc /app/bin/pandoc
    sources:
      - type: archive
        url: https://github.com/jgm/pandoc/releases/download/2.13/pandoc-2.13-linux-amd64.tar.gz
        sha256: 7404aa88a6eb9fbb99d9803b80170a3a546f51959230cc529c66a2ce6b950d4c

  - name: wkhtmltopdf
    buildsystem: simple
    build-commands:
      - cp -r local/* /app/
      - ls lib
    sources:
      - type: archive
        url: https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox-0.12.6-1.centos8.x86_64.rpm
        sha256: 5cc267d54fe3f144729f31432a165e028b689583c53cfee0a01d52644ab280d9
```

One of my dependencies is actually built and compiled within the flatpak container. (Given no `buildsystem` option on archives with makefiles in them, flatpak will automatically compile and install them)
```
  - name: enchant
    sources:
      - type: archive
        url: https://github.com/AbiWord/enchant/releases/download/v2.2.15/enchant-2.2.15.tar.gz
        sha256: 3b0f2215578115f28e2a6aa549b35128600394304bd79d6f28b0d3b3d6f46c03
```

So we are back to the dilemma of using source files or prebuilt binaries, you'll see later that the solution is a mix of both.

One option is just shipping a compiled version of my program in a flatpak. The issue is that flatpak's glibc is once again older than my main machine so *all* flatpak builds would have to be done in an Ubuntu VM, not ideal. Next up comes building from source in the flatpak. My plan for the process was this:

1. Get python and resource files in the flatpak
2. `pip install` all dependencies

Sounds simple enough, but flatpak didn't like me communicating with the internet while installing (I tried `share=network` on individual modules and it didn't work)
So, each python library had to be down as it's own python module with a link to the tar balls on pypi. Flatpak has a tool for such an occasion called [flatpak-pip-generator](https://github.com/flatpak/flatpak-builder-tools/tree/master/pip)
so the line in the flatpak yml
```yaml
- python3-requirements.json
```
is a pointer to a file actually containing all pip dependencies

```json
{
    "name": "python3-requirements",
    "buildsystem": "simple",
    "build-commands": [],
    "modules": [
        {
            "name": "python3-regex",
            "buildsystem": "simple",
            "build-commands": [
                "pip3 install --verbose --exists-action=i --no-index --find-links=\"file://${PWD}\" --prefix=${FLATPAK_DEST} \"regex\" --no-build-isolation"
            ],
            "sources": [
                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/38/3f/4c42a98c9ad7d08c16e7d23b2194a0e4f3b2914662da8bc88986e4e6de1f/regex-2021.4.4.tar.gz",
                    "sha256": "52ba3d3f9b942c49d7e4bc105bb28551c44065f139a65062ab7912bef10c9afb"
                }
            ]
        },
        {
            "name": "python3-num2words",
            "buildsystem": "simple",
            "build-commands": [
                "pip3 install --verbose --exists-action=i --no-index --find-links=\"file://${PWD}\" --prefix=${FLATPAK_DEST} \"num2words\" --no-build-isolation"
            ],
            "sources": [
                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/a2/55/8f8cab2afd404cf578136ef2cc5dfb50baa1761b68c9da1fb1e4eed343c9/docopt-0.6.2.tar.gz",
                    "sha256": "49b3a825280bd66b3aa83585ef59c4a8c82f2c8a522dbe754a8bc8d08c85c491"
                },
                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/33/db/76f1151a1b0cfad532d41021b77cd231495bf72c47618166f92dcdff2ebe/num2words-0.5.10.tar.gz",
                    "sha256": "37cd4f60678f7e1045cdc3adf6acf93c8b41bf732da860f97d301f04e611cc57"
                }
            ]
        },
        {
            "name": "python3-pypandoc",
            "buildsystem": "simple",
            "build-commands": [
                "pip3 install --verbose --exists-action=i --no-index --find-links=\"file://${PWD}\" --prefix=${FLATPAK_DEST} \"wheel\" --no-build-isolation",
                "pip3 install --verbose --exists-action=i --no-index --find-links=\"file://${PWD}\" --prefix=${FLATPAK_DEST} \"pypandoc\" --no-build-isolation"
            ],
            "sources": [
                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/ed/46/e298a50dde405e1c202e316fa6a3015ff9288423661d7ea5e8f22f589071/wheel-0.36.2.tar.gz",
                    "sha256": "e11eefd162658ea59a60a0f6c7d493a7190ea4b9a85e335b33489d9f17e0245e"
                },

                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/d6/b7/5050dc1769c8a93d3ec7c4bd55be161991c94b8b235f88bf7c764449e708/pypandoc-1.5.tar.gz",
                    "sha256": "14a49977ab1fbc9b14ef3087dcb101f336851837fca55ca79cf33846cc4976ff"
                }
            ]
        },
        {
            "name": "python3-pyenchant",
            "buildsystem": "simple",
            "build-commands": [
                "pip3 install --verbose --exists-action=i --no-index --find-links=\"file://${PWD}\" --prefix=${FLATPAK_DEST} \"pyenchant\" --no-build-isolation"
            ],
            "sources": [
                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/1f/d4/52869fe4ed24e373694c116adc00c82e5d92c747be4cbc97b24af43807f6/pyenchant-3.2.0.tar.gz",
                    "sha256": "6b97e9a9f132fa7c9029a6635d821ccad67d4980e68186d02c765b4256b6f152"
                }
            ]
        },
        {
            "name": "python3-PyPDF4",
            "buildsystem": "simple",
            "build-commands": [
                "pip3 install --verbose --exists-action=i --no-index --find-links=\"file://${PWD}\" --prefix=${FLATPAK_DEST} \"PyPDF4\" --no-build-isolation"
            ],
            "sources": [
                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/4f/1f/509b44850c475c101aa5b5c9b81755cedd363389d6fbb5c53be6fa915a61/PyPDF4-1.27.0.tar.gz",
                    "sha256": "7c932441146d205572f96254d53c79ea2c30c9e11df55a5cf87e056c7b3d7f89"
                }
            ]
        },
        {
            "name": "python3-setproctitle",
            "buildsystem": "simple",
            "build-commands": [
                "pip3 install --verbose --exists-action=i --no-index --find-links=\"file://${PWD}\" --prefix=${FLATPAK_DEST} \"setproctitle\" --no-build-isolation"
            ],
            "sources": [
                {
                    "type": "file",
                    "url": "https://files.pythonhosted.org/packages/a1/7f/a1d4f4c7b66f0fc02f35dc5c85f45a8b4e4a7988357a29e61c14e725ef86/setproctitle-1.2.2.tar.gz",
                    "sha256": "7dfb472c8852403d34007e01d6e3c68c57eb66433fb8a5c77b13b89a160d97df"
                }
            ]
        }
    ]
}
```

You'll notice that there is no mention of PySide or Qt. The *key* module that the app depends so heavily is not there, why? Well because both Qt python dependencies are not available as source files from pip. Pip hosts source files and wheels. All the above are source files. These are tar balls that pip will compile and build from source. Wheels are, once again, pre-built binaries. Flatpak doesn't include wheels by default since it wants everything to be as cross platform and cross architecture as possible.

I also wanted to be as platform agnostic as possible so I attempted to build my python bindings from source in the flatpak. Flatpak did not like this. Neither PySide nor PyQt would build and each threw different error during the compilation process. My best guess would by the sandbox does not contain some required system libraries but I am at a loss to know which ones. And C compilers are not the most helpful thing as your only point of reference for a system error.

### Lasers to the rescue
At this stage, I didn't actually know what python wheels were, I just assumed everything had to be built from source. I was desperate for answers after 3 days of debugging and staring at config files. I did something I very rarely do, I used the GitHub search function. To my amazement I found a solution.

The [Optical Metrology Group](https://www.physics.hu-berlin.de/en/qom) in the Physics department of Humboldt University Berlin [created a Linux application for locking onto spectroscopy lines](https://github.com/hermitdemschoenenleben/linien). Guess what, they made a flatpak!

I looked at there yaml, and I knew then and there that they had found a solution.

```yaml
  # we used to build pyqt on our own here, but this build caused trouble with the pyqtgraph widget being transparent on some systems.
  # Therefore, we just use the x86_64 wheel here...
  - name: pyqt-x86_64
    buildsystem: simple
    sources:
      - type: file
        url: https://files.pythonhosted.org/packages/4c/bb/7fce18fbe0275d7a3e069a306d8f4662c77eda30ec6780634fd4a7ee50ce/PyQt5-5.15.1-5.15.1-cp35.cp36.cp37.cp38.cp39-abi3-manylinux2014_x86_64.whl
        sha256: b1ea7e82004dc7b311d1e29df2f276461016e2d180e10c73805ace4376125ed9
      - type: file
        url: https://files.pythonhosted.org/packages/31/24/f887203677955ba4d5d4efe9176ac7ed2bf84efce8c243ab91e63183ad9e/PyQt5_sip-12.8.1-cp37-cp37m-manylinux1_x86_64.whl
        sha256: a1b8ef013086e224b8e86c93f880f776d01b59195bdfa2a8e0b23f0480678fec
    build-commands:
      - pip3 install PyQt*.whl --target=/app/lib/python3.7/site-packages
```

They had obviously also faced similar Qt-based problems. Their YAML led to me finding out about python wheels, how I could get PyQt in a flatpak, and how I could tell pip the right directory for a python installation (I had no idea what `--target` would do before this).

Thanks to some German physicists, my next project is back on track. Beady eyed people may notice some information in some of the config files I've posted that hint to what it could be. I did try and use PySide using the same wheel method but I got into some strange cyclical versioning thing since it required PySide an Shiboken2 to be installed at the same time. All that means is that my application will be open source out of a legal obligation, as well as an civil one.

*SIDE:* kerberos is also a requirement for PyQt to work in a flatpak, I have no idea why but it may have something to do with certain networking modules.

## Further Reading
Here are some useful sources of information I turned to while interacting with packaging systems on Linux

<https://www.loganasherjones.com/2018/05/using-flatpak-with-python/> 
:	An excellent and succinct guide that covers making non-qt python apps work in a flatpak

<https://docs.flatpak.org/en/latest/getting-started.html>
:	Information on what a flatpak is and what its parts consist of.

<https://docs.flatpak.org/en/latest/flatpak-builder-command-reference.html>
:	A good reference for what commands can be used in flatpak YAML

<https://github.com/flathub/io.github.hermitdemschoenenleben.linien/blob/8d89b1c7193602f0696f134e92ac1eba39986303/io.github.hermitdemschoenenleben.linien.yml#L18>
: 	The Humboldt University Berlin YAML

<https://opensource.com/article/18/9/how-build-rpm-packages>
:	A straightforward guide on how to make RPM packages
