# CKAD Exam Preparation

## First Step

1. Bash setup
    a) Open the terminal and edit the `.bashrc` file:

    ```bash
    vim ~/.bashrc
    ```

    b) Add the following lines to the end of the file:

    ```bash
    alias k=kubectl
    alias h=helm
    export do='--dry-run=client -o yaml'
    export now='--force --grace-period=0'
    
    source <(kubectl completion bash)
    complete -F __start_kubectl k
    source <(helm completion bash)
    ```

    c). Save and source it:

    ```bash
    source ~/.bashrc
    ```

2. Vim setup

    ```bash
    cat << 'EOF' >> ~/.vimrc
    set expandtab
    set tabstop=2
    set shiftwidth=2
    set autoindent
    set number
    EOF
    ```

3. Check the setup

    ```bash
    k get nodes
    ```
