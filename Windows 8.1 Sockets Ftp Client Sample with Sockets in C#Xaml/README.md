# Windows 8.1 Sockets: Ftp Client Sample with Sockets in C#/Xaml
## Requires
- Visual Studio 2013
## License
- MS-LPL
## Technologies
- Sockets
- FTP
- Windows.Networking.Sockets
## Topics
- Socket
- WinRT Sockets
- Windows 8.1
## Updated
- 03/30/2014
## Description

<h1>Introduction</h1>
<p><strong>UPDATE</strong>: 3.30.2014&nbsp;There was a bug in the retrieve function (with files bigger than 512bytes), this is fixed.</p>
<p>An FTP client begins by settings up a socket to the FTP server, authorize itself and then continues by sending instructions. All communication is done in plain text which makes the protocol both easy to understand and use.</p>
<p>Using the active and passive stream sockets, many ftp clients were imagined and developed in .net. However, the sockets implementatin in Windows Runtime (Windows Store Apps) has changed dramatically.</p>
<p>This sample gives at least access to the user the basic commands implementation using socket connections to an ftp server to execute ftp commands (e.g. STOR, RETR, MKD, RMD, DELE...)<em><br>
</em></p>
<h1><span>Running&nbsp;the Sample</span></h1>
<p>There are no third party libraries used in this sample.</p>
<p><strong>IMPORTANT : </strong>To&nbsp;run the tests project on <span style="text-decoration:underline">
local server</span> (i.e. If you are running a local ftp server), you have to&nbsp;disable the loopback check for&nbsp;the test&nbsp;project application. (<a title="How to enable loopback and troubleshoot network isolation (Windows Store Apps)" href="http://msdn.microsoft.com/en-us/library/windows/apps/hh780593.aspx" target="_blank">How
 to enable Loopback and troubleshoot network isolation</a>)</p>
<p style="text-align:justify"><img id="110630" src="http://i1.code.msdn.s-msft.com/windows-8-socketsftp-4fc23b33/image/file/110630/1/01-connectionerror%20(loopbackcheck).png" alt="" width="616" height="365" style="vertical-align:middle"></p>
<p>Easiest way to achieve this is to start debugging one of the unit tests, and use the Enable Loopback Utility to locate the test project and disable the loopback check.<em><br>
</em></p>
<p><span style="font-size:20px; font-weight:bold">Description</span></p>
<p><em>&nbsp;</em>The sample uses an abstraction layer (IFtpChannel<em>)</em> implemented in FtpStreamChannel and only created but not implemented in FtpWebChannel.</p>
<p>The FtpStreamChannel creates and connects to the sockets according to the commands the user wants to send which are organized in Messages namespace. (i.e. FtpCreateDirectoryRequest, FtpPassiveModeRequest...).</p>
<p>Each socket method is implemented in async pattern so as a developer you would need to be carefull about the asyc/awaits.</p>
<p>Forexample in order to open the initial socket, you would need to use the host name for the ftp server and &quot;21&quot; as the service name. Since we are not dealing with an FTPS or SFTP server, we can use the PlainSocket protection level to open our initial socket.</p>
<p>&nbsp;</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp"><span class="cs__keyword">public</span>&nbsp;async&nbsp;Task&lt;<span class="cs__keyword">bool</span>&gt;&nbsp;ConnectAsync(HostName&nbsp;host,&nbsp;<span class="cs__keyword">string</span>&nbsp;service)&nbsp;
{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;m_StreamSocket&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;StreamSocket();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">try</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;await&nbsp;m_StreamSocket.ConnectAsync(host,&nbsp;service,&nbsp;SocketProtectionLevel.PlainSocket).AsTask();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_IsConnected&nbsp;=&nbsp;<span class="cs__keyword">true</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">catch</span>&nbsp;(Exception&nbsp;ex)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;var&nbsp;socketError&nbsp;=&nbsp;SocketError.GetStatus(ex.HResult);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;TODO:&nbsp;log&nbsp;error</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;<span class="cs__keyword">false</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;<span class="cs__keyword">true</span>;&nbsp;
}</pre>
</div>
</div>
</div>
<div class="endscriptcode">When there is a need for expecting a reply for the command that we sent. For instance if we were to set the username create a directory, you can use the ExecuteAsync&lt;TReply&gt; method.</div>
<p>&nbsp;</p>
<p>&nbsp;</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp"><span class="cs__keyword">public</span>&nbsp;async&nbsp;Task&lt;T&gt;&nbsp;ExecuteAsync&lt;T&gt;(FtpRequest&nbsp;request)&nbsp;where&nbsp;T:FtpResponse&nbsp;
{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>(!IsConnected)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">throw</span>&nbsp;<span class="cs__keyword">new</span>&nbsp;FtpException(<span class="cs__string">&quot;Not&nbsp;Connected&quot;</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;await&nbsp;WriteLineAsync(Encoding,&nbsp;request.Command);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;await&nbsp;FlushAsync();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;await&nbsp;GetReplyAsync&lt;T&gt;();&nbsp;
}</pre>
</div>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
<p>&nbsp;</p>
<p>This will write the command you want to send in the opened socket and read the response from the server.</p>
<p>The higher level functions are just about using the ExecuteAsync function and expecting a reply.</p>
<p>On top of the FtpStreamChannel, there is an FtpClient implementation that implements higher level functions using the methods exposed by the IFtpChannel.</p>
<p>For instance, RetrieveFile method uses the streamed storage file approach, so that the files that are downloaded from the ftp server can easily be shared through the share contract.</p>
<p>&nbsp;</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;async&nbsp;Task&lt;StorageFile&gt;&nbsp;RetrieveFileAsync(<span class="cs__keyword">string</span>&nbsp;path)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(!(await&nbsp;FileExistsAsync(path)))&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">throw</span>&nbsp;<span class="cs__keyword">new</span>&nbsp;FileNotFoundException(<span class="cs__string">&quot;FTP&nbsp;file&nbsp;could&nbsp;not&nbsp;be&nbsp;retrieved&quot;</span>,&nbsp;path);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StorageFile&nbsp;resultantFile;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FtpResponse&nbsp;reply;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;A&nbsp;more&nbsp;efficient&nbsp;way,&nbsp;maybe&nbsp;a&nbsp;DataReader&nbsp;can&nbsp;be&nbsp;used&nbsp;here</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">using</span>&nbsp;(var&nbsp;stream&nbsp;=&nbsp;await&nbsp;OpenReadAsync(path))&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;var&nbsp;buffer&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;<span class="cs__keyword">byte</span>[<span class="cs__number">512</span>].AsBuffer();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;var&nbsp;resultingBuffer&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;<span class="cs__keyword">byte</span>[<span class="cs__number">0</span>];&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">while</span>&nbsp;(<span class="cs__keyword">true</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IBuffer&nbsp;readBuffer&nbsp;=&nbsp;await&nbsp;stream.ReadAsync(buffer,&nbsp;<span class="cs__number">512</span>,&nbsp;InputStreamOptions.Partial);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(readBuffer.Length&nbsp;==&nbsp;<span class="cs__number">0</span>)&nbsp;<span class="cs__keyword">break</span>;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;resultingBuffer&nbsp;=&nbsp;resultingBuffer.Concat(readBuffer.ToArray()).ToArray();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;resultantFile&nbsp;=&nbsp;await&nbsp;StorageFile.CreateStreamedFileAsync(path.GetFtpFileName(),&nbsp;async&nbsp;(fileStream)&nbsp;=&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;await&nbsp;fileStream.WriteAsync(resultingBuffer.AsBuffer());&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fileStream.FlushAsync();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fileStream.Dispose();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},&nbsp;<span class="cs__keyword">null</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(!(reply&nbsp;=&nbsp;await&nbsp;CloseDataStreamAsync(<span class="cs__keyword">null</span>)).Success)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">throw</span>&nbsp;<span class="cs__keyword">new</span>&nbsp;FtpCommandException(reply);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;resultantFile;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</pre>
</div>
</div>
</div>
<div class="endscriptcode">&nbsp;The unit tests project presents examples for different usages of FTP client.</div>
<p>&nbsp;</p>
<h1><span>Source Code Files</span></h1>
<ul>
<li>IFtpChannel.cs - Abstraction Layer for the ftp transport layer </li><li>FtpStreamChannel.cs - Implementation of the IFtpChannel </li><li>FtpClient - Ftp access using the IFtpChannel implementation </li></ul>
<h1>More Information</h1>
<p>Wait for the blog entry on <a href="http://canbilgin.wordpress.com/" target="_blank">
http://canbilgin.wordpress.com/</a></p>
<ul>
<li><a title="FTP Endeavor in WinRT - I" href="http://canbilgin.wordpress.com/2014/03/16/ftp-endeavor-in-winrt-i/" target="_blank">FTP Endeavor in WinRT - I</a>
</li><li><a href="http://canbilgin.wordpress.com/2014/03/30/ftp-endeavor-ii-upload-and-download-files/">FTP Endeavor in WinRT - II</a>
</li></ul>
<p>Project is inspired by <a title="http://netftp.codeplex.com/ " href="http://netftp.codeplex.com/ ">
http://netftp.codeplex.com/ </a></p>
<div class="mcePaste" id="_mcePaste" style="left:-10000px; top:0px; width:1px; height:1px; overflow:hidden">
</div>
