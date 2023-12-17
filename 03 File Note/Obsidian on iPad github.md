# Obsidian iPad syncing via iSH git

by Danny Quah, Jan 2022

This gist describes using Obsidian on iPad while syncing to other Obsidian platforms. The procedure uses `git` in `iSH` on `iOS`, and thus differs from using either `Obsidian Sync` or `Working Copy` as described in [Obsidian/iOS+app](https://help.obsidian.md/Obsidian/iOS+app).

(To be clear, `Obsidian` is one of my favourite Apps, and I'm all for supporting the team financially. Moreover, everything I've heard suggests the paid `Obsidian Sync` is excellent. However, I don't want my syncing processes to proliferate --- each service using a different client sync flow --- so I keep my systems minimal: just `syncthing` and `git`. After writing this I found an [Obsidian Forum writeup](https://forum.obsidian.md/t/mobile-sync-with-git-on-ios-for-free-using-ish/20861) which uses the same tools I do to achieve the same goal, but you'll want to read that with its accumulated contributions dispersed across the comments. So at least I was thinking about all this right :). Here, I just provide a linear, consistent run-through.)

I couldn't just begin using Obsidian on iPad directly as its instructions suggest, because I already have an Obsidian vault on my desktop. How do I bring that information over? iPad Obsidian does not let you open a folder on the iPad filesystem as a vault. And even if it did, how do you get your files into iPad's filesystem? Instead, iPad Obsidian opens up to give you only the option to create a new vault or to set up syncing through `Obsidian Sync` or `Working Copy`.

On desktop and Android, however, Obsidian is perfectly content letting you "Open folder as vault" acknowledging you likely already have existing folders of Markdown files. I wanted the same experience. True, iOS is relatively more closed than other platforms, but Obsidian elsewhere only needs a filesystem, folders, and Markdown files; nothing extra. On other platforms if I need to pop out and edit my Markdown files using something like `gvim`, I do so and come back into Obsidian again seamlessly, with all the changes I made reflected in my Obsidian interface. Surely iPad Obsidian must be able to leverage exactly the same structure that its same underlying code already uses elsewhere else.

It turns out this is possible. I do this using on the iPad a Linux command-line environment `iSH` and code-management system `git`; and on the desktop, related tools I already use there.

By "desktop" here I mean interchangeably your notebook, your desktop, your Android, ... basically every platform where Obsidian lives for you and on which you can run `git`, but that is not your iPad.

### [](https://gist.github.com/byally/e461704303240b464efaa7911de07061#ipad-preparation)iPad preparation

I'm going to describe how I got my Obsidian vault on my desktop into iPad Obsidian to use. If you need to do this in the opposite direction, the steps I describe will of course need be adjusted appropriately.

Initialize iPad for Obsidian use:

- (If needed) Install `iSH` on iPad.
- (If needed) Install `git` in `iSH`.
- (If needed) Install `doas` in `iSH`.

Because `git` will need to write into folders and files that are owned by the Obsidian process, not just your `iSH` login, you will need to invoke `git` everytime either as `root` or using `doas`.

### [](https://gist.github.com/byally/e461704303240b464efaa7911de07061#initialize-ios-obsidian-vault-and-ish-git-repo)Initialize iOS Obsidian vault and iSH git repo

1. (If needed) On github.com create `myWp` repo, and `git push` desktop's already-extant Obsidian vault folder up to it.
2. iPad>Obsidian > Create new vault `Wp`; exit Obsidian
3. iPad>`iSH`:  
    a. If needed, make mountpoint `mkdir -p /mnt/dq/Obsidian` (I make all my mountpoints contain my user login name `dq`, but it's unnecessary and you'll obviously want to adjust this for yourself. I just like things to look clean.)  
    b. `mount -t ios null /mnt/dq/Obsidian` (File picker appears, select Obsidian to associate with this mountpoint.). Mounting Obsidian rather than a specific vault means this method will work with multiple vaults. Subsequently just `cd`to the appropriate folder when in `iSH`.  
    c. Either as `root` or using `doas` (obviously replace the GitHub URL with your own):

```
cd /mnt/dq/Obsidian/Wp
git init
git remote add origin https://github.com/me/myWp.git
# Some users might have to do a "git add" and "git commit" before
# a local branch is available to rename. I didn't, so...
git branch -M main
git pull --rebase origin main # This might take a while
git push -u origin main
```

Open iPad>Obsidian and check that the contents of `myWp` are now available on the iPad's Obsidian vault `Wp`.

The sequence of steps 1.-3. above means three items are now in sync: (1) Obsidian vault folder on desktop; (2) Obsidian vault `Wp` folder on your iPad; (3) GitHub repo `myWp` containing the contents of your Obsidian vault folder.

### [](https://gist.github.com/byally/e461704303240b464efaa7911de07061#normal-work-cycle)Normal work cycle

Whenever you want to work on iPad>Obsidian:

1. Go to iPad>`iSH`; (if needed) `mount -t ios null /mnt/dq/Obsidian`; `cd /mnt/dq/Obsidian/Wp`; `git pull`.
2. Work away on iPad>Obsidian.
3. When done, go to `iSH`; (if needed) `mount -t ios null /mnt/dq/Obsidian` (choose Obsidian); `cd /mnt/dq/Obsidian/Wp` (or the appropriate other Obsidian vault sub-folder); `git add .; git commit -m "Message"; git push`.
4. (Optional) If I'm putting iPad aside for a while, I like to `umount /mnt/dq/Obsidian`, but that's up to you.

Whenever you want to work on desktop>Obsidian:

1. In a shell go to your Obsidian vault folder, and `git pull`.
2. Work away on desktop>Obsidian.
3. When done, in a shell go again to your Obsidian vault folder, and `git push`.

### [](https://gist.github.com/byally/e461704303240b464efaa7911de07061#additional-notes)Additional notes

1. The `.gitignore` in my Obsidian vault folders contain at least the line `.obsidian`. This folder can always be created as needed from the Obsidian app based on the data I provide in the vault, and I prefer each platform to have its own Obsidian customization. Moreover, the data I want to share are data I create myself, not information and cache that an app puts together, relatively invisibly, from my data and actions, and that might be finetuned to a specific filesystem and directory structure. So, yeah, `.obsidian` is in my `.gitignore`.
2. An alternative to using `iSH`, `mount -t ios`, and `git` is to use the (for these purposes) workalikes `a-Shell`, `pickFolder`, and `lg2`. These latter are all fast and good. Personally, however, I like having a full-blown linux distro on my iPad so I would choose `iSH` in any case. But, on the other hand, `git` on `iSH` continues to face issues of sporadic hanging on `commit` and `pull` (e.g. [ish-943](https://github.com/ish-app/ish/issues/943), [ish-1640](https://github.com/ish-app/ish/issues/1640)) so having `lg2` around or even as the main `git`-work client can be useful.
3. It would be excellent if iPad>Obsidian could just use any folder available in iPad's Files App. Right now, it can only pick up vaults created from within it in the first place. That is why I had first to create the empty vault before mounting Obsidian from `iSH`. The way things are now iPad's Files app can see my entire `iSH` filesystem. If Obsidian could too, I can then skip the `mount` and `umount` steps.


# 解决github无法push代码

[![林間飛鳥](https://picx.zhimg.com/v2-ddde8c200a86af46b3e02f66c3593cf7_l.jpg?source=172ae18b)](https://www.zhihu.com/people/nan-zhu-28-48)

[林間飛鳥](https://www.zhihu.com/people/nan-zhu-28-48)

接受所有依然爱这个世界

3 人赞同了该文章

- 将本地代码通过git push 推送到github仓库的时候，输入了密码但是依然无法push.

![](https://pic3.zhimg.com/80/v2-01d8a0873632483e81e04b263774bc1a_1440w.webp)

- 这段提示的内容是从2021年8月13日已经不支持密码的方式认证了，本文将介绍官方介绍的通过ssh的方式连接解决git push问题。

  

- 修改.git/config 文件提交通过ssh的方式而不是http

![](https://pic4.zhimg.com/80/v2-47c04166888f7555c423ac7798ad3bff_1440w.webp)

红框为固定内容，后面为对应的github仓库，后半段对应github仓库

![](https://pic4.zhimg.com/80/v2-cb8ad974772f087880521a575a14a333_1440w.webp)

![](https://pic1.zhimg.com/80/v2-43ea1e444569220dab0b09953d4fc080_1440w.webp)

  

- 如果没有设置过姓名和邮箱需要设置一下

```text
$ git config --global user.name "username"
$ git config --global user.email "xxxxx@xxx.com"
```

  

- 以下为官方提供的方法，首先生成ssh证书

```text
$ ssh-keygen -t ed25519 -C "your_email@example.com"
```

![](https://pic3.zhimg.com/80/v2-b3fc7607a5a17816b9db0981083dfc76_1440w.webp)

  

- 让ssh-agent 在后台运行

![](https://pic3.zhimg.com/80/v2-5801b4209af79e9f4ec821d9e6b5856e_1440w.webp)

- 将 SSH 私钥添加到 ssh-agent

```text
ssh-add ~/.ssh/id_ed25519
```

  

- 将生成的ssh密钥复制在剪切板

![](https://pic3.zhimg.com/80/v2-aab3302c817c7eb29e4b37e19e0cf22e_1440w.webp)

  

- 在任gitHub的右上角，点击您的个人资料照片，然后点击**设置**。  
    

![](https://pic3.zhimg.com/80/v2-e0bf13da46c34bbd5751ee105908f09a_1440w.webp)

- 在边栏的“访问”部分，单击**SSH 和 GPG 密钥**。

![](https://pic1.zhimg.com/80/v2-9418ffdd3daaa726bbe91ff4c8ee8d1c_1440w.webp)

  

- 单击"New SSH key"。

![](https://pic1.zhimg.com/80/v2-30a658dea262ebe592677606c3d24920_1440w.webp)

- Titel随便取即可，点击key的输入框ctrl+V粘贴即可  
    

![](https://pic1.zhimg.com/80/v2-4c98221e6129fb51c48252452c30aacc_1440w.webp)

- 单击**添加 SSH 密钥**。

![](https://pic3.zhimg.com/80/v2-812ee02e84a5d89af17e45ddecfaab62_1440w.webp)



https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys
# 生成新的 SSH 密钥并将其添加到 ssh-agent

## 本文内容

- [
    
    关于 SSH 密钥密码
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#about-ssh-key-passphrases)
- [
    
    生成新 SSH 密钥
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
- [
    
    将 SSH 密钥添加到 ssh-agent
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)
- [
    
    为硬件安全密钥生成新的 SSH 密钥
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key-for-a-hardware-security-key)

检查现有 SSH 密钥后，您可以生成新 SSH 密钥以用于身份验证，然后将其添加到 ssh-agent。

## Platform navigation

- [Mac](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=mac)
- [Windows](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=windows)
- [Linux](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux)

## [关于 SSH 密钥密码](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#about-ssh-key-passphrases)

可以使用 SSH（安全外壳协议）访问和写入 GitHub.com 上的存储库中的数据。 通过 SSH 进行连接时，使用本地计算机上的私钥文件进行身份验证。 有关详细信息，请参阅“[关于 SSH](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/about-ssh)”。

生成 SSH 密钥时，可以添加密码以进一步保护密钥。 每当使用密钥时，都必须输入密码。 如果密钥具有密码并且你不想每次使用密钥时都输入密码，则可以将密钥添加到 SSH 代理。 SSH 代理会管理 SSH 密钥并记住你的密码。

如果您还没有 SSH 密钥，则必须生成新 SSH 密钥用于身份验证。 如果不确定是否已经拥有 SSH 密钥，您可以检查现有密钥。 有关详细信息，请参阅“[检查现有 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)”。

如果要使用硬件安全密钥向 GitHub 验证，则必须为硬件安全密钥生成新的 SSH 密钥。 使用密钥对进行身份验证时，您必须将硬件安全密钥连接到计算机。 有关详细信息，请参阅 [OpenSSH 8.2 发行说明](https://www.openssh.com/txt/release-8.2)。

## [生成新 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)

可在本地计算机上生成新的 SSH 密钥。 生成密钥后，可以将公钥添加到你在 GitHub.com 上的帐户，以启用通过 SSH 进行 Git 操作的身份验证。

注意：GitHub 通过在 2022 年 3 月 15 日删除旧的、不安全的密钥类型来提高安全性。

自该日期起，不再支持 DSA 密钥 (`ssh-dss`)。 无法在 GitHub.com上向个人帐户添加新的 DSA 密钥。

2021 年 11 月 2 日之前带有 `valid_after` 的 RSA 密钥 (`ssh-rsa`) 可以继续使用任何签名算法。 在该日期之后生成的 RSA 密钥必须使用 SHA-2 签名算法。 一些较旧的客户端可能需要升级才能使用 SHA-2 签名。

1. 打开终端。
    
2. 粘贴以下文本，将示例中使用的电子邮件替换为 GitHub 电子邮件地址。
    
    ```shell
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```
    
    注意：如果你使用的是不支持 Ed25519 算法的旧系统，请使用以下命令：
    
    ```shell
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```
    
    这将以提供的电子邮件地址为标签创建新 SSH 密钥。
    
    ```shell
    > Generating public/private ALGORITHM key pair.
    ```
    
    当系统提示您“Enter a file in which to save the key（输入要保存密钥的文件）”时，可以按 Enter 键接受默认文件位置。 请注意，如果以前创建了 SSH 密钥，则 ssh-keygen 可能会要求重写另一个密钥，在这种情况下，我们建议创建自定义命名的 SSH 密钥。 为此，请键入默认文件位置，并将 id_ALGORITHM 替换为自定义密钥名称。
    
    ```shell
    > Enter a file in which to save the key (/home/YOU/.ssh/ALGORITHM):[Press enter]
    ```
    
3. 在提示符下，键入安全密码。 有关详细信息，请参阅“[使用 SSH 密钥密码](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases)”。
    
    ```shell
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```
    

## [将 SSH 密钥添加到 ssh-agent](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)

在向 ssh 代理添加新的 SSH 密钥以管理您的密钥之前，您应该检查现有 SSH 密钥并生成新的 SSH 密钥。 

1. 在后台启动 ssh 代理。
    
    ```shell
    $ eval "$(ssh-agent -s)"
    > Agent pid 59566
    ```
    
    根据您的环境，您可能需要使用不同的命令。 例如，在启动 ssh-agent 之前，你可能需要通过运行 `sudo -s -H` 根访问，或者可能需要使用 `exec ssh-agent bash` 或 `exec ssh-agent zsh` 运行 ssh-agent。
    
2. 将 SSH 私钥添加到 ssh-agent。
    
    如果使用其他名称创建了密钥或要添加具有其他名称的现有密钥，请将命令中的 ided25519 替换为私钥文件的名称。
    
    ```shell
    ssh-add ~/.ssh/id_ed25519
    ```
    
3. 将 SSH 公钥添加到 GitHub 上的帐户。 有关详细信息，请参阅“[新增 SSH 密钥到 GitHub 帐户](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)”。
    

## [为硬件安全密钥生成新的 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key-for-a-hardware-security-key)

如果您使用 macOS 或 Linux， 在生成新的 SSH 密钥之前，您可能需要更新 SSH 客户端或安装新的 SSH 客户端。 有关详细信息，请参阅“[错误：未知密钥类型](https://docs.github.com/zh/authentication/troubleshooting-ssh/error-unknown-key-type)”。

1. 将硬件安全密钥插入计算机。
    
2. 打开终端。
    
3. 粘贴以下文本，将示例中使用的电子邮件替换为与 GitHub 中帐户关联的电子邮件地址。
    
    ```shell
    ssh-keygen -t ed25519-sk -C "your_email@example.com"
    ```
    

注意：如果命令失败，并且你收到错误 `invalid format` 或 `feature not supported,`，则表明你可能在使用不支持 Ed25519 算法的硬件安全密钥。 请输入以下命令。

```shell
 ssh-keygen -t ecdsa-sk -C "your_email@example.com"
```

1. 出现提示时，请触摸硬件安全密钥上的按钮。
    
2. 当提示您“Enter a file in which to save the key（输入要保存密钥的文件）”时，按 Enter 接受默认文件位置。
    
    ```shell
    > Enter a file in which to save the key (/home/YOU/.ssh/id_ed25519_sk):[Press enter]
    ```
    
3. 当提示你输入密码时，请按 Enter。
    
    ```shell
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```
    
4. 将 SSH 公钥添加到 GitHub 上的帐户。 有关详细信息，请参阅“[新增 SSH 密钥到 GitHub 帐户](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)”。


# 成新的 SSH 密钥并将其添加到 ssh-agent

## 本文内容

- [
    
    关于 SSH 密钥密码
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#about-ssh-key-passphrases)
- [
    
    生成新 SSH 密钥
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
- [
    
    将 SSH 密钥添加到 ssh-agent
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)
- [
    
    为硬件安全密钥生成新的 SSH 密钥
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key-for-a-hardware-security-key)

检查现有 SSH 密钥后，您可以生成新 SSH 密钥以用于身份验证，然后将其添加到 ssh-agent。

## Platform navigation

- [Mac](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=mac)
- [Windows](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=windows)
- [Linux](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux)

## [关于 SSH 密钥密码](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#about-ssh-key-passphrases)

可以使用 SSH（安全外壳协议）访问和写入 GitHub.com 上的存储库中的数据。 通过 SSH 进行连接时，使用本地计算机上的私钥文件进行身份验证。 有关详细信息，请参阅“[关于 SSH](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/about-ssh)”。

生成 SSH 密钥时，可以添加密码以进一步保护密钥。 每当使用密钥时，都必须输入密码。 如果密钥具有密码并且你不想每次使用密钥时都输入密码，则可以将密钥添加到 SSH 代理。 SSH 代理会管理 SSH 密钥并记住你的密码。

如果您还没有 SSH 密钥，则必须生成新 SSH 密钥用于身份验证。 如果不确定是否已经拥有 SSH 密钥，您可以检查现有密钥。 有关详细信息，请参阅“[检查现有 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)”。

如果要使用硬件安全密钥向 GitHub 验证，则必须为硬件安全密钥生成新的 SSH 密钥。 使用密钥对进行身份验证时，您必须将硬件安全密钥连接到计算机。 有关详细信息，请参阅 [OpenSSH 8.2 发行说明](https://www.openssh.com/txt/release-8.2)。

## [生成新 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)

可在本地计算机上生成新的 SSH 密钥。 生成密钥后，可以将公钥添加到你在 GitHub.com 上的帐户，以启用通过 SSH 进行 Git 操作的身份验证。

注意：GitHub 通过在 2022 年 3 月 15 日删除旧的、不安全的密钥类型来提高安全性。

自该日期起，不再支持 DSA 密钥 (`ssh-dss`)。 无法在 GitHub.com上向个人帐户添加新的 DSA 密钥。

2021 年 11 月 2 日之前带有 `valid_after` 的 RSA 密钥 (`ssh-rsa`) 可以继续使用任何签名算法。 在该日期之后生成的 RSA 密钥必须使用 SHA-2 签名算法。 一些较旧的客户端可能需要升级才能使用 SHA-2 签名。

1. 打开终端。
    
2. 粘贴以下文本，将示例中使用的电子邮件替换为 GitHub 电子邮件地址。
    
    ```shell
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```
    
    注意：如果你使用的是不支持 Ed25519 算法的旧系统，请使用以下命令：
    
    ```shell
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```
    
    这将以提供的电子邮件地址为标签创建新 SSH 密钥。
    
    ```shell
    > Generating public/private ALGORITHM key pair.
    ```
    
    当系统提示您“Enter a file in which to save the key（输入要保存密钥的文件）”时，可以按 Enter 键接受默认文件位置。 请注意，如果以前创建了 SSH 密钥，则 ssh-keygen 可能会要求重写另一个密钥，在这种情况下，我们建议创建自定义命名的 SSH 密钥。 为此，请键入默认文件位置，并将 id_ALGORITHM 替换为自定义密钥名称。
    
    ```shell
    > Enter a file in which to save the key (/home/YOU/.ssh/ALGORITHM):[Press enter]
    ```
    
3. 在提示符下，键入安全密码。 有关详细信息，请参阅“[使用 SSH 密钥密码](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases)”。
    
    ```shell
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```
    

## [将 SSH 密钥添加到 ssh-agent](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)

在向 ssh 代理添加新的 SSH 密钥以管理您的密钥之前，您应该检查现有 SSH 密钥并生成新的 SSH 密钥。 

1. 在后台启动 ssh 代理。
    
    ```shell
    $ eval "$(ssh-agent -s)"
    > Agent pid 59566
    ```
    
    根据您的环境，您可能需要使用不同的命令。 例如，在启动 ssh-agent 之前，你可能需要通过运行 `sudo -s -H` 根访问，或者可能需要使用 `exec ssh-agent bash` 或 `exec ssh-agent zsh` 运行 ssh-agent。
    
2. 将 SSH 私钥添加到 ssh-agent。
    
    如果使用其他名称创建了密钥或要添加具有其他名称的现有密钥，请将命令中的 ided25519 替换为私钥文件的名称。
    
    ```shell
    ssh-add ~/.ssh/id_ed25519
    ```
    
3. 将 SSH 公钥添加到 GitHub 上的帐户。 有关详细信息，请参阅“[新增 SSH 密钥到 GitHub 帐户](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)”。
    

## [为硬件安全密钥生成新的 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key-for-a-hardware-security-key)

如果您使用 macOS 或 Linux， 在生成新的 SSH 密钥之前，您可能需要更新 SSH 客户端或安装新的 SSH 客户端。 有关详细信息，请参阅“[错误：未知密钥类型](https://docs.github.com/zh/authentication/troubleshooting-ssh/error-unknown-key-type)”。

1. 将硬件安全密钥插入计算机。
    
2. 打开终端。
    
3. 粘贴以下文本，将示例中使用的电子邮件替换为与 GitHub 中帐户关联的电子邮件地址。
    
    ```shell
    ssh-keygen -t ed25519-sk -C "your_email@example.com"
    ```
    

注意：如果命令失败，并且你收到错误 `invalid format` 或 `feature not supported,`，则表明你可能在使用不支持 Ed25519 算法的硬件安全密钥。 请输入以下命令。

```shell
 ssh-keygen -t ecdsa-sk -C "your_email@example.com"
```

1. 出现提示时，请触摸硬件安全密钥上的按钮。
    
2. 当提示您“Enter a file in which to save the key（输入要保存密钥的文件）”时，按 Enter 接受默认文件位置。
    
    ```shell
    > Enter a file in which to save the key (/home/YOU/.ssh/id_ed25519_sk):[Press enter]
    ```
    
3. 当提示你输入密码时，请按 Enter。
    
    ```shell
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```
    
4. 将 SSH 公钥添加到 GitHub 上的帐户。 有关详细信息，请参阅“[新增 SSH 密钥到 GitHub 帐户](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)”。
    

按 alt+up 激活

# 新增 SSH 密钥到 GitHub 帐户

## 本文内容

- [
    
    关于向帐户添加 SSH 密钥
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#about-addition-of-ssh-keys-to-your-account)
- [
    
    先决条件
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#prerequisites)
- [
    
    向你的帐户添加新的 SSH 密钥
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#adding-a-new-ssh-key-to-your-account)
- [
    
    延伸阅读
    
    ](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#further-reading)

要在 GitHub.com 上配置帐户以使用新的（或现有）SSH 密钥，还需要将密钥添加到帐户。

## Platform navigation

- [Mac](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?platform=mac)
- [Windows](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?platform=windows)
- [Linux](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?platform=linux)

## Tool navigation

- [GitHub CLI](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?tool=cli)
- [Web browser](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?tool=webui)

## [关于向帐户添加 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#about-addition-of-ssh-keys-to-your-account)

可以使用 SSH（安全外壳协议）访问和写入 GitHub.com 上的存储库中的数据。 通过 SSH 进行连接时，使用本地计算机上的私钥文件进行身份验证。有关详细信息，请参阅“[关于 SSH](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/about-ssh)”。

还可以使用 SSH 对提交和标记进行签名。 有关提交签名的详细信息，请参阅“[关于提交签名验证](https://docs.github.com/zh/authentication/managing-commit-signature-verification/about-commit-signature-verification)”。

生成 SSH 密钥对后，必须将公钥添加到 GitHub.com 以启用帐户的 SSH 访问。

## [先决条件](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#prerequisites)

在将新的 SSH 密钥添加到 GitHub.com 上的帐户之前，请完成以下步骤。

1. 检查现有 SSH 密钥。 有关详细信息，请参阅“[检查现有 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)”。
2. 生成新的 SSH 密钥，并将其添加到计算机的 SSH 代理。 有关详细信息，请参阅“[生成新的 SSH 密钥并将其添加到 ssh-agent](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)”。

## [向你的帐户添加新的 SSH 密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#adding-a-new-ssh-key-to-your-account)

可以添加 SSH 密钥并将其用于身份验证或提交签名，或同时用于这两个目的。 如果要使用相同的 SSH 密钥进行身份验证和签名，则需要将其上传两次。

为 GitHub.com 上的帐户添加新 SSH 身份验证密钥后，可以重新配置任何本地存储库以使用 SSH。 有关详细信息，请参阅“[管理远程仓库](https://docs.github.com/zh/get-started/getting-started-with-git/managing-remote-repositories#switching-remote-urls-from-https-to-ssh)”。

注意：GitHub 通过在 2022 年 3 月 15 日删除旧的、不安全的密钥类型来提高安全性。

自该日期起，不再支持 DSA 密钥 (`ssh-dss`)。 无法在 GitHub.com上向个人帐户添加新的 DSA 密钥。

2021 年 11 月 2 日之前带有 `valid_after` 的 RSA 密钥 (`ssh-rsa`) 可以继续使用任何签名算法。 在该日期之后生成的 RSA 密钥必须使用 SHA-2 签名算法。 一些较旧的客户端可能需要升级才能使用 SHA-2 签名。

1. 将 SSH 公钥复制到剪贴板。
    
    如果您的 SSH 公钥文件与示例代码不同，请修改文件名以匹配您当前的设置。 在复制密钥时，请勿添加任何新行或空格。
    

```shell
$ cat ~/.ssh/id_ed25519.pub
# Then select and copy the contents of the id_ed25519.pub file
# displayed in the terminal to your clipboard
```

提示：或者，你也可以找到隐藏的 `.ssh` 文件夹，在你最喜欢的文本编辑器中打开该文件，并将其复制到剪贴板。

1. 在任何页面的右上角，单击个人资料照片，然后单击“设置”。
    
    ![Screenshot of a user's account menu on GitHub. The menu item "Settings" is outlined in dark orange.](https://docs.github.com/assets/cb-45016/images/help/settings/userbar-account-settings-global-nav-update.png)
    
2. 在边栏的“访问”部分中，单击 “SSH 和 GPG 密钥”。
    
3. 单击“新建 SSH 密钥”或“添加 SSH 密钥” 。
    
4. 在 "Title"（标题）字段中，为新密钥添加描述性标签。 例如，如果使用的是个人笔记本电脑，则可以将此密钥称为“个人笔记本电脑”。
    
5. 选择密钥类型（身份验证或签名）。 有关提交签名的详细信息，请参阅“[关于提交签名验证](https://docs.github.com/zh/authentication/managing-commit-signature-verification/about-commit-signature-verification)”。
    
6. 在“密钥”字段中，粘贴公钥。
    
7. 单击“添加 SSH 密钥”。
    
8. 如果出现提示，请确认你的帐户是否拥有 GitHub 访问权限。 有关详细信息，请参阅“[Sudo 模式](https://docs.github.com/zh/authentication/keeping-your-account-and-data-secure/sudo-mode)”。