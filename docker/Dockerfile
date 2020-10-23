FROM ubuntu:focal

# General dependencies
RUN apt-get -qq update
RUN apt-get install -qq -y llvm-10 clang clang-10 libclang-10-dev cmake wget curl zip
RUN apt-get install -qq -y git

# Install lsif-clang
ENV LSIF_CLANG_REV=366659086a6ba2968c837179c17ff978cf3efc97
RUN git clone https://github.com/sourcegraph/lsif-clang.git
WORKDIR /lsif-clang
RUN git checkout $LSIF_CLANG_REV
RUN cmake -B build
RUN make -C build install
WORKDIR /

# src-cli
RUN curl -L https://sourcegraph.com/.api/src-cli/src_linux_amd64 -o /usr/local/bin/src
RUN chmod +x /usr/local/bin/src

#
# Build OpenCV
#
ENV PROJECT_REV=72dfd4846e184c480f27f137f3b078b538d1b017
RUN git clone --depth=10 https://github.com/opencv/opencv.git
WORKDIR /opencv
RUN git checkout $PROJECT_REV
# Create build directory
RUN mkdir -p build
WORKDIR /opencv/build
# Configure (generate compile_commands.json)
RUN cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ..
# Running build seems necessary, otherwise get opencl errors
RUN make -j8
RUN lsif-clang --project-root=/opencv compile_commands.json
RUN mv dump.lsif ..

ENV SRC_ACCESS_TOKEN=""
ENV SRC_ENDPOINT="https://sourcegraph.com"
WORKDIR /opencv
ENTRYPOINT src lsif upload
