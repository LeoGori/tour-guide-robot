ARG base_img
FROM $base_img

ARG CUDA_VERSION=12.1
# CUDA_INSTALLATION_MODE options are: deb_local, deb_network, runfile_local
# - deb_local: A local installer is self-contained. It is a large file that only needs to be downloaded from the internet once and can be installed on multiple systems. Local installers are the recommended type of installer with low-bandwidth internet connections, or where using a network installer is not possible (such as due to firewall restrictions).
# - deb_network: A network installer is a small installer client that will only download the required components during the installation. It is faster to download, but requires re-downloading each component with each new installation.
# - runfile_local: A local installer is self-contained. It is a large file that only needs to be downloaded from the internet once and can be installed on multiple systems. Local installers are the recommended type of installer with low-bandwidth internet connections, or where using a network installer is not possible (such as due to firewall restrictions).
ARG CUDA_INSTALLATION_MODE=deb_local

# # Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

RUN wget http://security.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2ubuntu0.1_amd64.deb
RUN sudo apt-get install ./libtinfo5_6.3-2ubuntu0.1_amd64.deb
RUN rm libtinfo5_6.3-2ubuntu0.1_amd64.deb

RUN CUDA_MAJOR=$(echo "$CUDA_VERSION" | cut -d. -f1) && CUDA_MINOR=$(echo "$CUDA_VERSION" | cut -d. -f2) && \
    # get the source code of the official CUDA download page based on the CUDA version
    curl -s "https://developer.nvidia.com/cuda-$CUDA_MAJOR-$CUDA_MINOR-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu" \
    # replace the unicode text with the actual characters: ", < and >,
    | sed 's/\\&quot;/"/g' | sed 's/\&quot;/"/g' | sed 's/\\u003c/</g' | sed 's/\\u003e/>/g' \
    # extract the metadata for the option Linux, x86_64, Ubuntu, *whichever version*, and local deb file, and get the last one (the assumption is that the last one is the latest Ubuntu version)
    | grep -o '"Linux/x86_64/Ubuntu/[0-9.]*/deb_local":{[^}]*' | tail -1 \
    # extract the first occurrence of the <pre> tag with the class cudaBashBlock
    | grep -oP '<pre class="cudaBashBlock">.*?</pre>' | head -1 \
    # grep the <span> tags and extract their content, i.e. the commands to download and install CUDA, and save them in a file
    | grep -o '<span class="cudaBash">[^<]*' | sed 's/<span class="cudaBash">//' > cuda-install.sh

RUN chmod +x cuda-install.sh
RUN ./cuda-install.sh
RUN rm cuda-install.sh
RUN rm cuda*

RUN echo "export PATH=/usr/local/cuda-$CUDA_VERSION/bin${PATH:+:${PATH}}" >> ~/.bashrc
RUN echo "export LD_LIBRARY_PATH=/usr/local/cuda-$CUDA_VERSION/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}" >> ~/.bashrc
RUN . ~/.bashrc
