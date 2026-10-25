# B-Trees

The most widely used structure for reading and writing database records.

- Keep key-value pairs sorted by key.

The log-structured index we saw earlier breaks the database into variable-size segments that are several megabytes or more in size. These segments are written once and are then immutable.

By contrast, **B-Trees** break the database into fixed-size blocks or pages and may overwrite a page in place. A page is typically 4 KiB in size.

<img width="353" height="230" alt="Screenshot 2026-10-24 at 5 26 49 PM" src="https://github.com/user-attachments/assets/4e94e278-61d0-48e5-927f-5e5e60fe512c" />

Each page has a page number, and it refers to another page on disk instead of in memory.

If all pages are stored in the same file, multiplying the page size by the page number gives you the byte offset in the file where the page is located.

We use page references to construct a tree of pages, as shown above.

One page contains several keys and references to child pages. Each child has a range of keys, and the keys between the references indicate where the boundaries between those ranges lie.

The number of references to child pages in one page of the B-Tree is called the **branching factor**.

To update the value of a key, you rewrite the entire page to the disk at the correct offset.

<img width="451" height="334" alt="Screenshot 2026-10-24 at 5 33 21 PM" src="https://github.com/user-attachments/assets/efbf2dbc-1af3-4a52-85ee-f081010de2c6" />

The splitting algorithm, which is used when a page gets full, keeps the tree balanced. A B-Tree with \(n\) keys always has a depth of \(O(\log n)\).

When splitting, we divide the page into two halves at the midpoint, then take the lowest key of the greater half and copy it to the parent to keep the references intact.
