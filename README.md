Signal for Android bruteforce backup decryption tool
=========================================

This repository contains improoved version of [this product](https://github.com/mossblaser/signal_for_android_decryption)

It is an (unofficial) tool which uses password list to decrypt backup files
produced by the [Signal](https://signal.org/) Android app.

What this tool is **not**:

* This tool does not export the backup data into a ready-to-use format, it
  simply decrypts the SQLite database and binary blobs within the backup file
  and leaves the rest up to you to work out.
* This tool does not support backup files produced by the iOS Signal app.
* This is not an official product from the Signal team. It depends on internal
  implementation details gleaned by reading the [source code of the Signal for
  Android app](https://github.com/signalapp/Signal-Android). While it appeared
  to work circa April 2023, this tool may break in the future.


Dependencies
------------

This tool is designed to run under Python 3.7+.

You can install the required Python dependencies as follows:

    $ pip install -r requirements.txt

Optionally, you may rebuild the [Protocol
Buffer](https://developers.google.com/protocol-buffers) Python module:

    $ protoc -I=. --python_out=. Backups.proto


Usage
-----

You can decrypt the backup file as follows:

```shell
$ python3 decrypt_backup.py -backup_file signal.backup -password_list rockyou.txt -output_directory decrypted_directory
```

The password list file may contain one password at a line and <b> the password should be 30 characters long only numbers </b> since signal produces such passwords by default.

If you have a large number of photographs and videos within any of
your groups the backup process can take several hours.

The backup will be decrypted into the directory you specified. The extracted
files are organised as follows:

* `database.sqlite`: A dump of the [SQLite](https://www.sqlite.org) database
  used by the signal app. This database is home to all of backed up messages
  and metadata aside from media files. The contents of this database are
  implementation defined and it is up to you to work out how to extract the
  information you want.
* `preferences.json`: A JSON file containing the 'preference' data encoded by
  the backup. Despite the existance of this file, most application preferences
  are stored in the SQLite database.
* `key_value.json`: A JSON file containing the 'key-value' data encoded by
  the backup.
* `attachments/*.bin`: Binary files attached to messages sent or received (e.g.
  photos and videos). These are named based on their `unique_id` as used in the
  database. To infer the correct file extension you'll need to lookup the MIME
  type in the database or guess based on the contents of the file.
* `stickers/*.bin`: Binary files containing sticker graphics used in chats. See
  also: attachments.
* `avatars/*.bin`: Binary files containing avatars images given to contacts and
  groups. See also: attachments.

A final obvious warning: all decrypted files are...unencrypted(!). You should
make sure to only perform decryption on a trusted device backed by sufficiently
secure storage for your needs.
