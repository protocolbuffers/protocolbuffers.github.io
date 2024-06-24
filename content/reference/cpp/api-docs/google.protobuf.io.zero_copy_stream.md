+++
title = "zero_copy_stream.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/io/zero_copy_stream.h&gt;<br>namespace <a href="#google.protobuf.io">google::protobuf::io</a></code></p><p>This file contains the <a href='#ZeroCopyInputStream'>ZeroCopyInputStream</a> and <a href='#ZeroCopyOutputStream'>ZeroCopyOutputStream</a> interfaces, which represent abstract I/O streams to and from which protocol buffers can be read and written. </p><p>For a few simple implementations of these interfaces, see <a href='google.protobuf.io.zero_copy_stream_impl'>zero_copy_stream_impl.h</a>.</p>

<p>These interfaces are different from classic I/O streams in that they try to minimize the amount of data copying that needs to be done. To accomplish this, responsibility for allocating buffers is moved to the stream object, rather than being the responsibility of the caller. So, the stream can return a buffer which actually points directly into the final data structure where the bytes are to be stored, and the caller can interact directly with that buffer, eliminating an intermediate copy operation.</p>

<p>As an example, consider the common case in which you are reading bytes from an array that is already in memory (or perhaps an mmap()ed file). With classic I/O streams, you would do something like: </p>

<pre>char buffer[BUFFER_SIZE];
input-&gt;Read(buffer, BUFFER_SIZE);
DoSomething(buffer, BUFFER_SIZE);</pre>

<p> Then, the stream basically just calls memcpy() to copy the data from the array into your buffer. With a <a href='#ZeroCopyInputStream'>ZeroCopyInputStream</a>, you would do this instead: </p>

<pre>const void* buffer;
int size;
input-&gt;Next(&amp;buffer, &amp;size);
DoSomething(buffer, size);</pre>

<p> Here, no copy is performed. The input stream returns a pointer directly into the backing array, and the caller ends up reading directly from it.</p>

<p>If you want to be able to read the old-fashion way, you can create a <a href='google.protobuf.io.coded_stream#CodedInputStream'>CodedInputStream</a> or <a href='google.protobuf.io.coded_stream#CodedOutputStream'>CodedOutputStream</a> wrapping these objects and use their ReadRaw()/WriteRaw() methods. These will, of course, add a copy step, but Coded*Stream will handle buffering so at least it will be reasonably efficient.</p>

<p><a href='#ZeroCopyInputStream'>ZeroCopyInputStream</a> example: </p>

<pre>// Read in a file and print its contents to stdout.
int fd = open("myfile", O_RDONLY);
ZeroCopyInputStream* input = new FileInputStream(fd);

const void* buffer;
int size;
while (input-&gt;Next(&amp;buffer, &amp;size)) {
  cout.write(buffer, size);
}

delete input;
close(fd);</pre>

<p><a href='#ZeroCopyOutputStream'>ZeroCopyOutputStream</a> example: </p>

<pre>// Copy the contents of "infile" to "outfile", using plain read() for
// "infile" but a ZeroCopyOutputStream for "outfile".
int infd = open("infile", O_RDONLY);
int outfd = open("outfile", O_WRONLY);
ZeroCopyOutputStream* output = new FileOutputStream(outfd);

void* buffer;
int size;
while (output-&gt;Next(&amp;buffer, &amp;size)) {
  int bytes = read(infd, buffer, size);
  if (bytes &lt; size) {
    // Reached EOF.
    output-&gt;BackUp(size - bytes);
    break;
  }
}

delete output;
close(infd);
close(outfd);</pre>

<table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr><tr><td><div><code><a href="#ZeroCopyInputStream">ZeroCopyInputStream</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Abstract interface similar to an input stream but designed to minimize copying. </div></td></tr><tr><td><div><code><a href="#ZeroCopyOutputStream">ZeroCopyOutputStream</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Abstract interface similar to an output stream but designed to minimize copying. </div></td></tr></table><h2 id="ZeroCopyInputStream">class ZeroCopyInputStream</h2><p><code>#include &lt;<a href="#">google/protobuf/io/zero_copy_stream.h</a>&gt;<br>namespace <a href="#google.protobuf.io">google::protobuf::io</a></code></p><p>Abstract interface similar to an input stream but designed to minimize copying. </p><p>Known subclasses:</p><ul><li><code><a href="google.protobuf.io.zero_copy_stream_impl_lite#ArrayInputStream">ArrayInputStream</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl#ConcatenatingInputStream">ConcatenatingInputStream</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl_lite#CopyingInputStreamAdaptor">CopyingInputStreamAdaptor</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl#FileInputStream">FileInputStream</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl#IstreamInputStream">IstreamInputStream</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl_lite#LimitingInputStream">LimitingInputStream</a></code></li></ul><table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="ZeroCopyInputStream.ZeroCopyInputStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>ZeroCopyInputStream</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual </code></td><td style="border-left-width: 0px"id="ZeroCopyInputStream.~ZeroCopyInputStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>~ZeroCopyInputStream</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual bool</code></td><td style="border-left-width: 0px"id="ZeroCopyInputStream.Next"><div style="padding-left: 16px; text-indent: -16px"><code><b>Next</b>(const void ** data, int * size)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Obtains a chunk of data from the stream.  <a href="#ZeroCopyInputStream.Next.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual void</code></td><td style="border-left-width: 0px"id="ZeroCopyInputStream.BackUp"><div style="padding-left: 16px; text-indent: -16px"><code><b>BackUp</b>(int count)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Backs up a number of bytes, so that the next call to <a href='#ZeroCopyInputStream.Next'>Next()</a> returns data again that was already returned by the last call to <a href='#ZeroCopyInputStream.Next'>Next()</a>.  <a href="#ZeroCopyInputStream.BackUp.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual bool</code></td><td style="border-left-width: 0px"id="ZeroCopyInputStream.Skip"><div style="padding-left: 16px; text-indent: -16px"><code><b>Skip</b>(int count)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Skips a number of bytes.  <a href="#ZeroCopyInputStream.Skip.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual int64_t</code></td><td style="border-left-width: 0px"id="ZeroCopyInputStream.ByteCount"><div style="padding-left: 16px; text-indent: -16px"><code><b>ByteCount</b>() const  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Returns the total number of bytes read since this object was created. </div></td></tr></table> <hr><h3 id="ZeroCopyInputStream.Next.details"><code>virtual bool ZeroCopyInputStream::Next(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const void ** data,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int * size)  = 0</code></h3><div style="margin-left: 16px"><p>Obtains a chunk of data from the stream. </p><p>Preconditions:</p>
<ul>
  <li>"size" and "data" are not NULL.</li>
</ul>
<p>Postconditions:</p>
<ul>
  <li>If the returned value is false, there is no more data to return or an error occurred. All errors are permanent.</li>
  <li>Otherwise, "size" points to the actual number of bytes read and "data" points to a pointer to a buffer containing these bytes.</li>
  <li>Ownership of this buffer remains with the stream, and the buffer remains valid only until some other method of the stream is called or the stream is destroyed.</li>
  <li>It is legal for the returned buffer to have zero size, as long as repeatedly calling <a href='#ZeroCopyInputStream.Next'>Next()</a> eventually yields a buffer with non-zero size. </li>
</ul>
</div> <hr><h3 id="ZeroCopyInputStream.BackUp.details"><code>virtual void ZeroCopyInputStream::BackUp(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int count)  = 0</code></h3><div style="margin-left: 16px"><p>Backs up a number of bytes, so that the next call to <a href='#ZeroCopyInputStream.Next'>Next()</a> returns data again that was already returned by the last call to <a href='#ZeroCopyInputStream.Next'>Next()</a>. </p><p>This is useful when writing procedures that are only supposed to read up to a certain point in the input, then return. If <a href='#ZeroCopyInputStream.Next'>Next()</a> returns a buffer that goes beyond what you wanted to read, you can use <a href='#ZeroCopyInputStream.BackUp'>BackUp()</a> to return to the point where you intended to finish.</p>
<p>Preconditions:</p>
<ul>
  <li>The last method called must have been <a href='#ZeroCopyInputStream.Next'>Next()</a>.</li>
  <li>count must be less than or equal to the size of the last buffer returned by <a href='#ZeroCopyInputStream.Next'>Next()</a>.</li>
</ul>
<p>Postconditions:</p>
<ul>
  <li>The last "count" bytes of the last buffer returned by <a href='#ZeroCopyInputStream.Next'>Next()</a> will be pushed back into the stream. Subsequent calls to <a href='#ZeroCopyInputStream.Next'>Next()</a> will return the same data again before producing new data. </li>
</ul>
</div> <hr><h3 id="ZeroCopyInputStream.Skip.details"><code>virtual bool ZeroCopyInputStream::Skip(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int count)  = 0</code></h3><div style="margin-left: 16px"><p>Skips a number of bytes. </p><p>Returns false if the end of the stream is reached or some input error occurred. In the end-of-stream case, the stream is advanced to the end of the stream (so <a href='#ZeroCopyInputStream.ByteCount'>ByteCount()</a> will return the total size of the stream). </p>
</div><h2 id="ZeroCopyOutputStream">class ZeroCopyOutputStream</h2><p><code>#include &lt;<a href="#">google/protobuf/io/zero_copy_stream.h</a>&gt;<br>namespace <a href="#google.protobuf.io">google::protobuf::io</a></code></p><p>Abstract interface similar to an output stream but designed to minimize copying. </p><p>Known subclasses:</p><ul><li><code><a href="google.protobuf.io.zero_copy_stream_impl_lite#ArrayOutputStream">ArrayOutputStream</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl_lite#CopyingOutputStreamAdaptor">CopyingOutputStreamAdaptor</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl#OstreamOutputStream">OstreamOutputStream</a></code></li><li><code><a href="google.protobuf.io.zero_copy_stream_impl_lite#StringOutputStream">StringOutputStream</a></code></li></ul><table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="ZeroCopyOutputStream.ZeroCopyOutputStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>ZeroCopyOutputStream</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual </code></td><td style="border-left-width: 0px"id="ZeroCopyOutputStream.~ZeroCopyOutputStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>~ZeroCopyOutputStream</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual bool</code></td><td style="border-left-width: 0px"id="ZeroCopyOutputStream.Next"><div style="padding-left: 16px; text-indent: -16px"><code><b>Next</b>(void ** data, int * size)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Obtains a buffer into which data can be written.  <a href="#ZeroCopyOutputStream.Next.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual void</code></td><td style="border-left-width: 0px"id="ZeroCopyOutputStream.BackUp"><div style="padding-left: 16px; text-indent: -16px"><code><b>BackUp</b>(int count)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Backs up a number of bytes, so that the end of the last buffer returned by <a href='#ZeroCopyOutputStream.Next'>Next()</a> is not actually written.  <a href="#ZeroCopyOutputStream.BackUp.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual int64_t</code></td><td style="border-left-width: 0px"id="ZeroCopyOutputStream.ByteCount"><div style="padding-left: 16px; text-indent: -16px"><code><b>ByteCount</b>() const  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Returns the total number of bytes written since this object was created. </div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual bool</code></td><td style="border-left-width: 0px"id="ZeroCopyOutputStream.WriteAliasedRaw"><div style="padding-left: 16px; text-indent: -16px"><code><b>WriteAliasedRaw</b>(const void * data, int size)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Write a given chunk of data to the output.  <a href="#ZeroCopyOutputStream.WriteAliasedRaw.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual bool</code></td><td style="border-left-width: 0px"id="ZeroCopyOutputStream.AllowsAliasing"><div style="padding-left: 16px; text-indent: -16px"><code><b>AllowsAliasing</b>() const</code></div></td></tr></table> <hr><h3 id="ZeroCopyOutputStream.Next.details"><code>virtual bool ZeroCopyOutputStream::Next(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;void ** data,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int * size)  = 0</code></h3><div style="margin-left: 16px"><p>Obtains a buffer into which data can be written. </p><p>Any data written into this buffer will eventually (maybe instantly, maybe later on) be written to the output.</p>
<p>Preconditions:</p>
<ul>
  <li>"size" and "data" are not NULL.</li>
</ul>
<p>Postconditions:</p>
<ul>
  <li>If the returned value is false, an error occurred. All errors are permanent.</li>
  <li>Otherwise, "size" points to the actual number of bytes in the buffer and "data" points to the buffer.</li>
  <li>Ownership of this buffer remains with the stream, and the buffer remains valid only until some other method of the stream is called or the stream is destroyed.</li>
  <li>Any data which the caller stores in this buffer will eventually be written to the output (unless <a href='#ZeroCopyOutputStream.BackUp'>BackUp()</a> is called).</li>
  <li>It is legal for the returned buffer to have zero size, as long as repeatedly calling <a href='#ZeroCopyOutputStream.Next'>Next()</a> eventually yields a buffer with non-zero size. </li>
</ul>
</div> <hr><h3 id="ZeroCopyOutputStream.BackUp.details"><code>virtual void ZeroCopyOutputStream::BackUp(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int count)  = 0</code></h3><div style="margin-left: 16px"><p>Backs up a number of bytes, so that the end of the last buffer returned by <a href='#ZeroCopyOutputStream.Next'>Next()</a> is not actually written. </p><p>This is needed when you finish writing all the data you want to write, but the last buffer was bigger than you needed. You don't want to write a bunch of garbage after the end of your data, so you use <a href='#ZeroCopyOutputStream.BackUp'>BackUp()</a> to back up.</p>
<p>Preconditions:</p>
<ul>
  <li>The last method called must have been <a href='#ZeroCopyOutputStream.Next'>Next()</a>.</li>
  <li>count must be less than or equal to the size of the last buffer returned by <a href='#ZeroCopyOutputStream.Next'>Next()</a>.</li>
  <li>The caller must not have written anything to the last "count" bytes of that buffer.</li>
</ul>
<p>Postconditions:</p>
<ul>
  <li>The last "count" bytes of the last buffer returned by <a href='#ZeroCopyOutputStream.Next'>Next()</a> will be ignored. </li>
</ul>
</div> <hr><h3 id="ZeroCopyOutputStream.WriteAliasedRaw.details"><code>virtual bool ZeroCopyOutputStream::WriteAliasedRaw(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const void * data,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int size)</code></h3><div style="margin-left: 16px"><p>Write a given chunk of data to the output. </p><p>Some output streams may implement this in a way that avoids copying. Check AllowsAliasing() before calling <a href='#ZeroCopyOutputStream.WriteAliasedRaw'>WriteAliasedRaw()</a>. It will GOOGLE_CHECK fail if <a href='#ZeroCopyOutputStream.WriteAliasedRaw'>WriteAliasedRaw()</a> is called on a stream that does not allow aliasing.</p>
<p>NOTE: It is caller's responsibility to ensure that the chunk of memory remains live until all of the data has been consumed from the stream. </p>
</div>
