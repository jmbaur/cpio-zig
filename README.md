# cpio-zig

A Zig library for writing cpio archives in the "new ASCII" (`newc`, magic `070701`) format, the same format the Linux kernel expects for an initramfs. All entries are written with uid/gid 0 and mtime 0 for reproducibility.

## Usage

Archive a whole directory tree:

```zig
const std = @import("std");
const CpioArchive = @import("cpio");

pub fn archiveDir(io: std.Io, gpa: std.mem.Allocator, root: []const u8, out: *std.Io.Writer) !void {
    var arena = std.heap.ArenaAllocator.init(gpa);
    defer arena.deinit();

    var dir = try std.Io.Dir.cwd().openDir(io, root, .{ .iterate = true });
    defer dir.close(io);

    var archive = try CpioArchive.init(out);
    try CpioArchive.walkDirectory(io, &arena, root, &archive, &dir);
    try archive.finalize();
}
```

Or building an archive manually:

```zig
var archive = try CpioArchive.init(out);
try archive.addDirectory("etc", 0o755);
try archive.addSymlink("bin/sh", "busybox");
try archive.addFile(io, "init", file, size, permissions);
try archive.addCharacterDevice("dev/console", 0o600, 5, 1);
try archive.addBlockDevice("dev/sda", 0o660, 8, 0);
try archive.addFifo("run/initctl", 0o600);
try archive.addSocket("run/log", 0o666);
try archive.finalize(); // writes the TRAILER!!! entry, pads, and flushes
```
