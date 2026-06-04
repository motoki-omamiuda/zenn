# zenn
account: [@mamech](https://zenn.dev/mamech)

## how to use
* install zenn cli
    ```bash
    npm init
    npm install zenn-cli
    ```

* update zenn cli
    ```bash
    npm install zenn-cli@latest
    ```

* create a new article
    ```bash
    npx zenn new:article
    # 記事のURLの一部となるslugを指定して作成することもできます。
    npx zenn new:article --slug my-awesome-article
    ```

* preview
    ```bash
    npx zenn preview
    ```

> [!WARNING]
Even if you delete a file from the directory, the article will not be removed.
To delete an article, please do so from the [dashboard](https://zenn.dev/dashboard).

##  references
* [install zenn cli](https://zenn.dev/zenn/articles/install-zenn-cli)
* [how to manage articles and books with zenn cli](https://zenn.dev/zenn/articles/zenn-cli-guide)
