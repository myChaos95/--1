# rsync

## 语法

`$ rsync options source detination`

### 1. 本地到本地

`$ rsync -zvr /var /opt/ /root/temp`

### 2. 本地到源端 

`$ rsync -azv /root/temp/ chaos@192.168.0.1:/home/chaos/temp/`

### 3. 本地到服务器 

`$ rsync -avz /root/temp/ chaos@192.168.0.1::/home/chaos/temp/`

## 不带任何选项

`$ rsync main.c machineB:/home/userB`

1. 只要目的端的文件内容和源端不一致，就会触发数据同步，rsync 会确保两边的文件内容一样。
2. 但是 rsync 不会同步文件的 "modify time"，凡是有数据同步的文件，目的端文件的 "modify time" 总是会被修改为最新的时间。
3. rsync 的不会改变目的端文件的权限。
4. rsync 需要对源文件有读权限，并且对目的端文件有写权限

## -t选项

`$ rsync -t main.c machineB:/home/userB`

1. rsync 会将源文件的 "modify time" 同步到目标机器
2. 同步前会先比较两边文件的时间戳和文件大小，如果一致，则认为两边文件一样，就对此文件不再采取更新动作了。
3. 但是如果目的端文件的 时间戳、文件大小 完全一样，但是内容不一致时，rsync 是发现不了的， 使用 -i 来解决。

## -i 选项

`$ rsync -i main.c machineB:/home/userB`

1. 会让 rsync 挨个文件发起数据同步。
2. 保证了数据的一致性，代价是速度变慢

## -v选项

` $ rsync -v main.c machineB:/home/userB`  `verbose`

1. 让 rsync 输出更多的信息，增加的 v 越多，就可以获取更多的日志信息。

## -z选项

`$ rsync -z main.c machineB:/home/userB`

1. 会把发送的数据先进行压缩再传输。

## -r选项

`$ rsync -r folder machineB:/home/userB`   `recursive`

`skiping directory folder `

1. rsync 不会帮你同步文件夹，使用 -r 同步文件夹。

## -p选项

`$ rsync -p main.c machineB:/home/userB`

1. 让目的端保持和源端的权限一致。

## -u选项

`$ rsync -u root/main/ machineB:/home/main`

1. 如果文件在目的地上修改过，那么不要覆盖它

## -d选项

`$ rsync -d root/main/ machineB:/home/main`

1. 只会递归同步目录树，目录中的文件不会同步

