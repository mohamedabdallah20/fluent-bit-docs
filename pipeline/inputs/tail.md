# Tail

The _Tail_ input plugin lets you monitor text files. Its behavior is similar to the `tail -f` shell command.

The plugin reads every matched file in the `Path` pattern. For every new line found (separated by a newline character (`\n`), it generates a new record. Optionally, you can use a database file so the plugin can have a history of tracked files and a state of offsets. This helps resume a state if the service is restarted.

## Configuration parameters

The plugin supports the following configuration parameters:

| Key | Description | Default |
| :-- | :---------- | :------ |
| `Buffer_Chunk_Size` | Set the initial buffer size to read file data. This value is used to increase buffer size. The value must be according to the [Unit Size](../../administration/configuring-fluent-bit/unit-sizes.md) specification. | `32k` |
| `Buffer_Max_Size` | Set the limit of the buffer size per monitored file. When a buffer needs to be increased, this value is used to restrict the memory buffer growth. If reading a file exceeds this limit, the file is removed from the monitored file list. The value must be according to the [Unit Size](../../administration/configuring-fluent-bit/unit-sizes.md) specification. | `32k` |
| `Path` | Pattern specifying a specific log file or multiple ones through the use of common wildcards. Allows multiple patterns separated by commas. | _none_ |
| `Path_Key` | If enabled, it appends the name of the monitored file as part of the record. The value assigned becomes the key in the map. | _none_ |
| `Exclude_Path` | Set one or multiple shell patterns separated by commas to exclude files matching certain criteria, For example, `Exclude_Path *.gz,*.zip`. | _none_ |
| `Offset_Key` | If enabled, Fluent Bit appends the offset of the current monitored file as part of the record. The value assigned becomes the key in the map. | _none_ |
| `Read_from_Head` | For new discovered files on start (without a database offset/position), read the content from the head of the file, not tail. | `False` |
| `Refresh_Interval` | The interval of refreshing the list of watched files in seconds. | `60` |
| `Rotate_Wait` | Specify the number of extra time in seconds to monitor a file once is rotated in case some pending data is flushed. | `5` |
| `Ignore_Older` | Ignores files older than `ignore_older`. Supports `m`, `h`, `d` (minutes, hours, days) syntax.  | Read all. |
| `Skip_Long_Lines` | When a monitored file reaches its buffer capacity due to a very long line (`Buffer_Max_Size`), the default behavior is to stop monitoring that file. `Skip_Long_Lines` alter that behavior and instruct Fluent Bit to skip long lines and continue processing other lines that fit into the buffer size. | `Off` |
| `Skip_Empty_Lines` | Skips empty lines in the log file from any further processing or output. | `Off` |
| `DB` | Specify the database file to keep track of monitored files and offsets. | _none_ |
| `DB.sync` | Set a default synchronization (I/O) method. This flag affects how the internal SQLite engine do synchronization to disk, for more details about each option see [the SQLite documentation](https://www.sqlite.org/pragma.html#pragma_synchronous). Most scenarios will be fine with `normal` mode. If you need full synchronization after every write operation set `full` mode. `full` has a high I/O performance cost. Values: `Extra`, `Full`, `Normal`, `Off`. | `normal` |
| `DB.locking` | Specify that the database will be accessed only by Fluent Bit. Enabling this feature helps increase performance when accessing the database but restricts externals tool from querying the content. | `false` |
| `DB.journal_mode` | Sets the journal mode for databases (`WAL`). Enabling `WAL` provides higher performance. `WAL` isn't compatible with shared network file systems. | `WAL` |
| `DB.compare_filename` | This option determines whether to review both `inode` and `filename` when retrieving stored file information from the database. `true` verifies both `inode` and `filename`, while `false` checks only the `inode`. To review the `inode` and `filename` in the database, refer [see `keep_state`](#tailing-files-keeping-state). | `false` |
| `Mem_Buf_Limit` | Set a memory limit that Tail plugin can use when appending data to the engine. If the limit is reached, it will be paused. When the data is flushed it resumes. | _none_ |
| `Exit_On_Eof` | When reading a file will exit as soon as it reach the end of the file. Used for bulk load and tests. | `false` |
| `Parser` | Specify the name of a parser to interpret the entry as a structured message. | _none_ |
| `Key` | When a message is unstructured (no parser applied), it's appended as a string under the key name `log`. This option lets you define an alternative name for that key. | `log` |
| `Inotify_Watcher` | Set to `false` to use file stat watcher instead of `inotify`. | `true` |
| `Tag` | Set a tag with `regexextract` fields that will be placed on lines read. For example, `kube.<namespace_name>.<pod_name>.<container_name>.<container_id>`. Tag expansion is supported: if the tag includes an asterisk (`*`), that asterisk will be replaced with the absolute path of the monitored file, with slashes replaced by dots. See [Workflow of Tail + Kubernetes Filter](../filters/kubernetes.md#workflow-of-tail--kubernetes-filter). | _none_ |
| `Tag_Regex` | Set a regular expression to extract fields from the filename. For example: `(?<pod_name>[a-z0-9](?:[-a-z0-9]*[a-z0-9])?(?:\\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?<namespace_name>[^_]+)_(?<container_name>.+)-(?<container_id>[a-z0-9]{64})\.log$`. | _none_ |
| `Static_Batch_Size` | Set the maximum number of bytes to process per iteration for the monitored static files (files that already exist upon Fluent Bit start). | `50M` |
| `File_Cache_Advise` | Set the `posix_fadvise` in `POSIX_FADV_DONTNEED` mode. This reduces the usage of the kernel file cache. This option is ignored if not running on Linux. | `On` |
| `Threaded` | Indicates whether to run this input in its own [thread](../../administration/multithreading.md#inputs). | `false` |
| `Unicode.Encoding` | Set the Unicode character encoding of the file data. This parameter requests two-byte aligned chunk and buffer sizes. If data is not aligned for two bytes, Fluent Bit will use two-byte alignment automatically to avoid character breakages on consuming boundaries. Supported values: `UTF-16LE`, `UTF-16BE`, and `auto`. | _none_ |
| `Generic.Encoding` | Set the encoding which the origin of character encoding. Currently, ShiftJIS, UHC, GBK, GB18030, Big5, Win866, Win874, Win1250, Win1251, Win1252, Win2513, Win1254, Win1255, Win1256 are supported. | _none_ |

{% hint style="info" %}
If the database parameter `DB` isn't specified, by default the plugin reads each target file from the beginning. This might cause unwanted behavior. For example, when a line is bigger than `Buffer_Chunk_Size` and `Skip_Long_Lines` isn't turned on, the file will be read from the beginning of each `Refresh_Interval` until the file is rotated.
{% endhint %}

{% hint style="info" %}
The `Unicode.Encoding` parameter is dependent on the simdutf library, which is itself dependent on C++ version 11 or later. In environments that use earlier versions of C++, the `Unicode.Encoding` parameter will fail.

Additionally, the `auto` setting for `Unicode.Encoding` isn't supported in all cases, and can make mistakes when it tries to guess the correct encoding. For best results, use either the `UTF-16LE` or `UTF-16BE` setting if you know the encoding type of the target file.
{% endhint %}

## Monitor a large number of files

To monitor a large number of files, you can increase the `inotify` settings in your Linux environment by modifying the following `sysctl` parameters:

```text
sysctl fs.inotify.max_user_watches=LIMIT1
sysctl fs.inotify.max_user_instances=LIMIT2
```

Replace _`LIMIT1`_ and _`LIMIT2`_ with the integer values of your choosing. Higher values raise your `inotify` limit accordingly.

These changes revert upon reboot unless you write them to the appropriate `inotify.conf` file. Writing to the file ensures the settings persist across reboots. The specific filename can vary depending on how you built and installed Fluent Bit. For example, to write changes to a file named `fluent-bit_fs_inotify.conf`, run the following commands:

```shell
mkdir -p /etc/sysctl.d
echo fs.inotify.max_user_watches = LIMIT1 >> /etc/sysctl.d/fluent-bit_fs_inotify.conf
echo fs.inotify.max_user_instances = LIMIT2 >> /etc/sysctl.d/fluent-bit_fs_inotify.conf
```

Replace _`LIMIT1`_ and _`LIMIT2`_ with the integer values of your choosing.

You can also provide a custom systemd configuration file that overrides the default systemd settings for Fluent Bit. This override file must be located at `/etc/systemd/system/fluent-bit.service.d/override.conf` or `/etc/systemd/system/fluent-bit.service.d/override.yaml` depending on the configuration you choose. For example, you can add one of these snippets to your override file to raise the number of files that the Tail plugin can monitor:

{% tabs %}
{% tab title="override.yaml" %}

```yaml
service:
    limitnofile: LIMIT
```

{% endtab %}
{% tab title="override.conf" %}

```text
[Service]
LimitNOFILE=LIMIT
```

{% endtab %}
{% endtabs %}

Replace _`LIMIT`_ with the integer value of your choosing.

If you don't already have an override file, you can use the following command to create one in the correct directory:

```shell
systemctl edit fluent-bit.service
```

## Multiline support

Fluent Bit 1.8 and later supports multiline core capabilities for the Tail input plugin. Fluent Bit supports the both the old and new configuration mechanisms. To avoid breaking changes, users are encouraged to use the latest one. The two mechanisms are:

- Multiline Core
- Old Multiline

### Multiline Core (v1.8)

The new multiline core is exposed by the following configuration:

| Key | Description |
| :--- | :--- |
| `multiline.parser` | Specify one or multiple [Multiline Parser definitions](../../administration/configuring-fluent-bit/multiline-parsing.md) to apply to the content. |

[Multiline Parser](../../administration/configuring-fluent-bit/multiline-parsing.md) provides built-in configuration modes. When using a new `multiline.parser` definition, you must disable the old configuration from your tail section like:

- `parser`
- `parser_firstline`
- `parser_N`
- `multiline`
- `multiline_flush`
- `docker_mode`

### Multiline and containers (v1.8)

If you are running Fluent Bit to process logs coming from containers like Docker or CRI, you can use the built-in modes. This helps reassemble multiline messages originally split by Docker or CRI:

{% tabs %}
{% tab title="fluent-bit.yaml" %}

```yaml
pipeline:
    inputs:
        - name: tail
          path: /var/log/containers/*.log
          multiline.parser: docker, cri
```

{% endtab %}
{% tab title="fluent-bit.conf" %}

```text
[INPUT]
    name              tail
    path              /var/log/containers/*.log
    multiline.parser  docker, cri
```

{% endtab %}
{% endtabs %}

The two options separated by a comma mean Fluent Bit will try each parser in the list in order, applying the first one that matches the log.

It will use the first parser which has a `start_state` that matches the log.

For example, it will first try `docker`, and if `docker` doesn't match, it will then try `cri`.

### Old multiline configuration parameters

For the old multiline configuration, the following options exist to configure the handling of multiline logs:

| Key | Description | Default |
| :--- | :--- | :--- |
| `Multiline` | If enabled, the plugin will try to discover multiline messages and use the proper parsers to compose the outgoing messages. When this option is enabled the Parser option isn't used. | `Off` |
| `Multiline_Flush` | Wait period time in seconds to process queued multiline messages. | `4` |
| `Parser_Firstline` | Name of the parser that matches the beginning of a multiline message. The regular expression defined in the parser must include a group name (named `capture`), and the value of the last match group must be a string. | _none_ |
| `Parser_N` | Optional. Extra parser to interpret and structure multiline entries. This option can be used to define multiple parsers. For example, `Parser_1 ab1`, `Parser_2 ab2`, `Parser_N abN`. | _none_ |

### Old Docker mode configuration parameters

Docker mode exists to recombine JSON log lines split by the Docker daemon due to its line length limit. To use this feature, configure the tail plugin with the corresponding parser and then enable Docker mode:

| Key | Description | Default |
| :--- | :--- | :--- |
| `Docker_Mode` | If enabled, the plugin will recombine split Docker log lines before passing them to any parser. This mode can't be used at the same time as Multiline. | `Off` |
| `Docker_Mode_Flush` | Wait period time in seconds to flush queued unfinished split lines. | `4` |
| `Docker_Mode_Parser` | Specify an optional parser for the first line of the Docker multiline mode. The parser name to be specified must be registered in the `parsers.conf` file. | _none_ |

## Get started

To tail text or log files, you can run the plugin from the command line or through the configuration file.

### Command line

From the command line you can let Fluent Bit parse text files with the following options:

```shell
fluent-bit -i tail -p path=/var/log/syslog -o stdout
```

### Configuration file

Append the following in your main configuration file:

{% tabs %}
{% tab title="fluent-bit.yaml" %}

```yaml
pipeline:
    inputs:
      - name: tail
        path: /var/log/syslog

    outputs:
      - stdout:
        match: *
```

{% endtab %}
{% tab title="fluent-bit.conf" %}

```text
[INPUT]
    Name    tail
    Path    /var/log/syslog

[OUTPUT]
    Name   stdout
    Match  *
```

{% endtab %}
{% endtabs %}

### Old multiline example

When using multiline configuration you need to first specify `Multiline On` in the configuration and use the `Parser_Firstline` and additional parser parameters `Parser_N` if needed.

For example, you might be trying to read the following Java stack trace as a single event:

```text
Dec 14 06:41:08 Exception in thread "main" java.lang.RuntimeException: Something has gone wrong, aborting!
    at com.myproject.module.MyProject.badMethod(MyProject.java:22)
    at com.myproject.module.MyProject.oneMoreMethod(MyProject.java:18)
    at com.myproject.module.MyProject.anotherMethod(MyProject.java:14)
    at com.myproject.module.MyProject.someMethod(MyProject.java:10)
    at com.myproject.module.MyProject.main(MyProject.java:6)
```

Specify a `Parser_Firstline` parameter that matches the first line of a multiline event. When a match is made, Fluent Bit reads all future lines until another match with `Parser_Firstline` is made.

In this case you can use the following parser, which extracts the time as `time` and the remaining portion of the multiline as `log`.

{% tabs %}
{% tab title="fluent-bit.yaml" %}

```yaml
parsers:
    - name: multiline
      format: regex
      regex: '/(?<time>[A-Za-z]+ \d+ \d+\:\d+\:\d+)(?<message>.*)/'
      time_key: time
      time_format: '%b %d %H:%M:%S'
```

{% endtab %}
{% tab title="fluent-bit.conf" %}

```text
[PARSER]
    Name multiline
    Format regex
    Regex /(?<time>[A-Za-z]+ \d+ \d+\:\d+\:\d+)(?<message>.*)/
    Time_Key  time
    Time_Format %b %d %H:%M:%S
```

{% endtab %}
{% endtabs %}

To further parse the entire event, you can add additional parsers with `Parser_N` where N is an integer. The final Fluent Bit configuration looks like the following:

{% tabs %}
{% tab title="fluent-bit.yaml" %}

```yaml
parsers:
    - name: multiline
      format: regex
      regex: '/(?<time>[A-Za-z]+ \d+ \d+\:\d+\:\d+)(?<message>.*)/'
      time_key: time
      time_format: '%b %d %H:%M:%S'

pipeline:
    inputs:
        - name:  tail
          multiline: on
          read_from_head: true
          parser_firstline: multiline
          path: /var/log/java.log

    outputs:
        - name: stdout
          match: '*'
```

{% endtab %}
{% tab title="fluent-bit.conf" %}

```text
# Note this is generally added to parsers.conf and referenced in [SERVICE]
[PARSER]
    Name multiline
    Format regex
    Regex /(?<time>[A-Za-z]+ \d+ \d+\:\d+\:\d+)(?<message>.*)/
    Time_Key  time
    Time_Format %b %d %H:%M:%S

[INPUT]
    Name             tail
    Multiline        On
    Parser_Firstline multiline
    Path             /var/log/java.log

[OUTPUT]
    Name             stdout
    Match            *
```

{% endtab %}
{% endtabs %}

The output will be as follows:

```text
[0] tail.0: [1607928428.466041977, {"message"=>"Exception in thread "main" java.lang.RuntimeException: Something has gone wrong, aborting!
    at com.myproject.module.MyProject.badMethod(MyProject.java:22)
    at com.myproject.module.MyProject.oneMoreMethod(MyProject.java:18)
    at com.myproject.module.MyProject.anotherMethod(MyProject.java:14)
    at com.myproject.module.MyProject.someMethod(MyProject.java:10)", "message"=>"at com.myproject.module.MyProject.main(MyProject.java:6)"}]
```

## Tailing files keeping state

The _tail_ input plugin a feature to save the state of the tracked files, is strongly suggested you enabled this. For this purpose the `db` property is available.

For example:

```shell
fluent-bit -i tail -p path=/var/log/syslog -p db=/path/to/logs.db -o stdout
```

When Fluent Bit runs, the database file `_/path/to/logs.db` will be created. This database is backed by SQLite3. If you are interested in exploring the content, you can open it with the SQLite client tool:

```shell
sqlite3 tail.db
```

Which returns results like:

```text
-- Loading resources from /home/myusername/.sqliterc

SQLite version 3.14.1 2016-08-11 18:53:32
Enter ".help" for usage hints.
sqlite> SELECT * FROM in_tail_files;
id     name                              offset        inode         created
-----  --------------------------------  ------------  ------------  ----------
1      /var/log/syslog                   73453145      23462108      1480371857
sqlite>
```

When exploring, ensure Fluent Bit isn't hard working on the database file, or you will see `Error: database is locked` messages.

### Formatting SQLite

By default, SQLite client tools don't format the columns in a human-readable way. To explore the `in_tail_files` table, you can create a configuration file in `~/.sqliterc` with the following content:

```text
.headers on
.mode column
.width 5 32 12 12 10
```

## SQLite and Write Ahead Logging

Fluent Bit keeps the state or checkpoint of each file through using a SQLite database file. If the service is restarted, it can continue consuming files from it last checkpoint position (offset). The default options set are enabled for high performance and corruption-safe.

The SQLite journal mode enabled is `Write Ahead Log` or `WAL`. This allows improved performance of read and write operations to disk. When enabled, you will see in your file system additional files being created, consider the following configuration statement:

{% tabs %}
{% tab title="fluent-bit.yaml" %}

```yaml
pipeline:
    inputs:
      - name:  tail
        path: /var/log/containers/*.log
        db: test.db
```

{% endtab %}
{% tab title="fluent-bit.conf" %}

```text
[INPUT]
    name    tail
    path    /var/log/containers/*.log
    db      test.db
```

{% endtab %}
{% endtabs %}

The previous configuration enables a database file called `test.db` and in the same path for that file SQLite will create two additional files:

- `test.db-shm`
- `test.db-wal`

These two files support the `WAL` mechanism that helps improve performance and reduce the number system calls required. The `-wal` file refers to the file that stores the new changes to be committed. At some point the `WAL` file transactions are moved back to the real database file. The `-shm` file is a shared-memory type to allow concurrent-users to the `WAL` file.

### Write Ahead Logging (WAL) and memory usage

The `WAL` mechanism gives higher performance but also might increase the memory usage by Fluent Bit. Most of this usage comes from the memory mapped and cached pages. In some cases you might see that memory usage keeps a bit high giving the impression of a memory leak, but actually isn't relevant unless you want your memory metrics back to normal. Fluent Bit 1.7.3 introduced the new option `db.journal_mode` that sets the journal mode for databases. By default this mode is set to `WAL (Write-Ahead Logging)`. Allowed configurations for `db.journal_mode` are `DELETE | TRUNCATE | PERSIST | MEMORY | WAL | OFF` .

## File rotation

File rotation is properly handled, including `logrotate`'s `copytruncate` mode.

{% hint style="warning" %}

While file rotation is handled, there are risks of potential log loss when using `logrotate` with `copytruncate` mode:

- Race conditions: logs can be lost in the brief window between copying and truncating the file.
- Backpressure: if Fluent Bit is under backpressure, logs might be dropped if `copyttruncate` occurs before they can be processed and sent.
- See `logroate man page`: there is a very small time slice between copying the file and truncating it, so some logging data might be lost.
- Final note: the `Path` patterns can't match the rotated files. Otherwise, the rotated file would be read again and lead to duplicate records.

{% endhint %}

## Character Encoding Conversion

This feature allows Fluent Bit to convert logs from various character encodings into the standard UTF-8 format.
This is crucial for processing logs from systems, especially Windows, that use legacy or non-UTF-8 encodings.
Proper conversion ensures that your log data is correctly parsed, indexed, and searchable.

### When to Use This Feature

You should use this feature if your log files or messages are not in UTF-8 and you are seeing garbled or incorrectly rendered characters.
This is common in environments that use:

* Modern Windows applications that log in UTF-16.

* Legacy Windows systems with applications that use traditional code pages (e.g., ShiftJIS, GBK, Win1252).

### Configuration Parameters

To enable encoding conversion, you will use one of the following two parameters within an input plugin configuration.

1. `Unicode.Encoding`

Use this parameter for high-performance conversion of UTF-16 encoded logs to UTF-8. This method utilizes modern processor features (SIMD instructions) to accelerate the conversion process, making it highly efficient.

* Use Case: Ideal for logs coming from modern Windows environments that default to UTF-16.
* Supported Values:
    * UTF-16LE (Little-Endian)
    * UTF-16BE (Big-Endian)

2. `Generic.Encoding`

Use this parameter to convert from a wide variety of other character encodings, particularly legacy Windows code pages.

* Use Case: Essential for logs from older systems or applications configured for specific regions, common in East Asia and Eastern Europe.
* Supported Values: You can use any of the names or aliases listed below.

### East Asian Encodings
* `ShiftJIS` (Aliases: `SJIS`, `CP932`, `Windows-31J`)
* `GB18030`
* `GBK`: (Alias: `CP936`)
* `UHC` (Unified Hangul Code): (Aliases: `CP949` and `Windows-949`)
* `Big5`: (Alias: `CP950`)

### Windows (ANSI) Encodings
* `Win1250` (Central European): (Alias: `CP1250`)
* `Win1251` (Cyrillic): (Alias: `CP1251`)
* `Win1252` (Western European / Latin): (Alias: `CP1252`)
* `Win1253` (Greek): (Alias: `CP1253`)
* `Win1254` (Turkish): (Alias: `CP1254`)
* `Win1255` (Hebrew): (Alias: `CP1255`)
* `Win1256` (Arabic): (Alias: `CP1256`)

### DOS (OEM) Encodings
* `Win866` (Cyrillic - DOS): (Alias: `CP866`)
* `Win874` (Thai): (Alias: `CP874`)

### Configuration Example

Here is an example of how to use `Generic.Encoding` with the Tail input plugin to read a log file encoded in ShiftJIS.

{% tabs %}
{% tab title="fluent-bit.yaml" %}

```yaml
pipeline:
    inputs:
      - name:  tail
        path: /var/log/containers/*.log
        generic.encoding:    ShiftJIS
```

{% endtab %}
{% tab title="fluent-bit.conf" %}

```text
[INPUT]
    Name                tail
    Path                C:\path\to\your\sjis.log
    Generic.Encoding    ShiftJIS
```

{% endtab %}
{% endtabs %}
