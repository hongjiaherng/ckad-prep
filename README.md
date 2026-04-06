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
    set expandtab # Use spaces instead of tabs
    set tabstop=2 # Number of spaces that a <Tab> in the file counts for
    set shiftwidth=2 # Number of spaces to use for each step of (auto)indent
    set autoindent # Copy indent from current line when starting a new line
    set number # Show line numbers
    EOF
    ```

3. Check the setup

    ```bash
    k get nodes
    ```
