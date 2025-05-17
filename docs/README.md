# Solidity Language Docs

## how run locally the documentation? environment setup

1. install
   1. [python](https://www.python.org/downloads/)
   2. [sphinx](https://www.sphinx-doc.org/en/master/usage/installation.html)
2.
    ```sh
    cd docs
    ./docs.sh
    ```
   * | "_build/", generate htmls
3. `python3 -m http.server -d _build/html --cgi 8080`
4. | browser,
   1. http://localhost:8080
