# wxasync
asyncio support for wxpython

This library defines two things: WxAsyncApp, and AsyncBind. 
Just create a WxAsyncApp instead of a wx.App, and use AsyncBind when you want
to bind an event to a coroutine. 
You then start the application using loop.run_until_complete(app.MainLoop()).

Below is a simple example:

```python
import wx
from wxasync import AsyncBind, WxAsyncApp
import asyncio
from asyncio.events import get_event_loop


class TestFrame(wx.Frame):
    def __init__(self, parent=None):
        super(TestFrame, self).__init__(parent)
        vbox = wx.BoxSizer(wx.VERTICAL)
        button1 =  wx.Button(self, label="Submit")
        self.edit =  wx.StaticText(self, style=wx.ALIGN_CENTRE_HORIZONTAL|wx.ST_NO_AUTORESIZE)
        vbox.Add(button1, 2, wx.EXPAND|wx.ALL)
        vbox.AddStretchSpacer(1)
        vbox.Add(self.edit, 1, wx.EXPAND|wx.ALL)
        self.SetSizer(vbox)
        self.Layout()
        AsyncBind(wx.EVT_BUTTON, self.async_callback, button1)

    async def async_callback(self, event):
        self.edit.SetLabel("Button clicked")
        await asyncio.sleep(1)
        self.edit.SetLabel("Working")
        await asyncio.sleep(1)
        self.edit.SetLabel("Completed")


app = WxAsyncApp()
frame = TestFrame()
frame.Show()
app.SetTopWindow(frame)
loop = get_event_loop()
loop.run_until_complete(app.MainLoop())
```

## Performance

wxasync does GUI message polling every 5ms, and runs the asyncio message loop the rest of the time.
This gives pretty good results

Below is view of the performances (on windows Core I7-7700K 4.2Ghz):

| Scenario      |Latency  |  Latency (max throughput)| Throughput(msg/s) |
| ------------- |--------------|---------------------------------|-------------|
| asyncio only (for reference)  |0ms             |17ms                               |571 325|
| wx only (for reference)       |0ms             |19ms                               |94 591|
| wxasync (GUI) | 5ms            |19ms                               |52 304|
| wxasync (GUI+asyncio)| 5ms GUI / 0ms asyncio |24ms GUI / 12ms asyncio |40 302 GUI + 134 000 asyncio|


The performance tests are included in the 'test' directory.
