## Test env

- M1 Max 64GB MacBook Pro
- macOS 14.2.1 (23C71)

## How to install

### Disable System Integrity Protection(SIP)

**You need to disable System Integrity Protection(SIP).** <https://docs.ros.org/en/humble/Installation/Alternatives/macOS-Development-Setup.html#disable-system-integrity-protection-sip>

### Install brew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Execute the command mentioned at the end of the brew installation here**

```bash
brew install \
    asio assimp bison bullet cmake console_bridge cppcheck \
    cunit eigen freetype graphviz opencv openssl orocos-kdl pcre poco \
    pyqt5 python qt@5 sip spdlog tinyxml tinyxml2 wget
```

```bash
brew uninstall --ignore-dependencies python@3.12 qt6
```

```bash
brew install pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> $HOME/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> $HOME/.zshrc
echo 'eval "$(pyenv init -)"' >> $HOME/.zshrc
pyenv install 3.11.8
pyenv global 3.11.8
pyenv local 3.11.8
source $HOME/.zshrc
```

```bash
python3.11 -m pip install -U pip
python3.11 -m pip install --global-option=build_ext \
       --global-option="-I$(brew --prefix graphviz)/include/" \
       --global-option="-L$(brew --prefix graphviz)/lib/" \
       pygraphviz
python3.11 -m pip install -U \
      argcomplete catkin_pkg colcon-common-extensions coverage \
      cryptography empy==3.3.4 flake8 flake8-blind-except==0.1.1 flake8-builtins \
      flake8-class-newline flake8-comprehensions flake8-deprecated \
      flake8-docstrings flake8-import-order flake8-quotes \
      importlib-metadata lark==1.1.1 lxml matplotlib mock mypy==0.931 netifaces \
      nose pep8 psutil pydocstyle pydot pygraphviz pyparsing==2.4.7 \
      pytest-mock rosdep rosdistro setuptools==59.6.0 vcstool
```

```bash
git clone https://github.com/hanchangyo/ros2_humble_m1
mkdir ${HOME}/ros2_humble_m1/src
cd ${HOME}/ros2_humble_m1/
vcs import src < ros2.repos
```

```bash
patch -l < patches/ros2_console_bridge_vendor.patch
patch -l < patches/ros2_rviz_ogre_vendor.patch
patch -l < patches/ros_visualization_rqt_bag.patch
```

```bash
export CMAKE_PREFIX_PATH=$CMAKE_PREFIX_PATH:$(brew --prefix qt@5)
export PATH=$PATH:$(brew --prefix qt@5)/bin
export COLCON_EXTENSION_BLOCKLIST=colcon_core.event_handler.desktop_notification

# Get the Python executable path
export PYTHON_EXECUTABLE=$(which python)
echo "Python executable: $PYTHON_EXECUTABLE"

# Get the Python include directory
export PYTHON_INCLUDE_DIR=$(python -c "from sysconfig import get_paths; print(get_paths()['include'])")
echo "Python include directory: $PYTHON_INCLUDE_DIR"

# Get the Python library directory
export PYTHON_LIB_DIR=$(python -c "import sysconfig; print(sysconfig.get_config_var('LIBDIR'))")
echo "Python library directory: $PYTHON_LIB_DIR"

# Get the actual Python library file (e.g., libpython3.11.dylib)
export PYTHON_LIBRARY_FILE=$(python -c "import sysconfig; print(sysconfig.get_config_var('LDLIBRARY'))")
export PYTHON_LIBRARY="$PYTHON_LIB_DIR/$PYTHON_LIBRARY_FILE"
echo "Python library: $PYTHON_LIBRARY"

# Export necessary environment variables for CMake
export CMAKE_LIBRARY_PATH="$PYTHON_LIB_DIR:$CMAKE_LIBRARY_PATH"

python3.11 -m colcon build --symlink-install --packages-skip-by-dep qt_gui_cpp --packages-skip qt_gui_cpp --cmake-args \
            -DBUILD_TESTING=OFF \
            -DTHIRDPARTY=FORCE \
            -DCMAKE_BUILD_TYPE=Release \
            -DPYTHON_EXECUTABLE="$PYTHON_EXECUTABLE" \
            -DPYTHON_INCLUDE_DIR="$PYTHON_INCLUDE_DIR" \
            -DPYTHON_LIBRARY="$PYTHON_LIBRARY" \
            "$@" \
            -Wno-dev
```

## Write to .zshrc

```bash
echo "source ${HOME}/ros2_humble_m1/install/setup.zsh" >> ${HOME}/.zshrc
echo "export ROS_VERSION=2" >> ${HOME}/.zshrc
echo "export ROS_PYTHON_VERSION=3" >> ${HOME}/.zshrc
echo "export ROS_DISTRO=humble" >> ${HOME}/.zshrc
echo "export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp" >> ${HOME}/.zshrc
source ${HOME}/.zshrc
```

## Quick run

```bash
# terminal 1
ros2 run demo_nodes_cpp talker
```

```bash
# terminal 2
ros2 run demo_nodes_py listener
```
