# leymonton
import discord
from discord.ext import commands
from youtube_dl import YoutubeDL

BOT = commands.Bot(command_prefix='404')

class music_cog(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

        self.is_playing = False
        self.ctx = ''
        self.loop = False
        self.music_queue = []
        self.YDL_OPTIONS = {
          'format' : 'bestaudio/best',
          'postprocessors' : [{
            'key' : 'FFmpegExtractAudio',
            'preferredcodec' : 'mp3',
            'preferredquality' : '192',
          }], 'noplaylist':'True'}
        self.FFMPEG_OPTIONS = {'before_options':'-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5', 'options':'-vn'}

        self.vc = ''

    def search_yt(self, item):
        with YoutubeDL(self.YDL_OPTIONS) as ydl:
            try:
                info = ydl.extract_info(f'ytsearch:{item}', download=False)['entries'][0]
            except Exception:
                return False
        return {'source': info['formats'][0]['url'], 'title': info['title']}

    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False
	        def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)    def play_next(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']
            self.music_queue.pop(0)


            self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())


        else:
            self.is_playing = False

    async def play_music(self):
        if len(self.music_queue)>0:
            self.is_playing = True

            m_url = self.music_queue[0][0]['source']

            if self.vc == '':
              self.vc = await self.music_queue[0][1].connect()
            elif self.music_queue[0][1] != self.vc:
              self.vc.disconnect()
              self.vc = await self.music_queue[0][1].connect()

            await self.ctx.send('Now playing : ' + self.music_queue[0][0]['title'])

            try:
              self.ctx.voice_client.play(discord.FFmpegPCMAudio(m_url, **self.FFMPEG_OPTIONS), after = lambda e : self.play_next())
            except:
              await self.ctx.send(self.vc)
            try:
              self.music_queue.pop(0)
            except:
              await self.ctx.send(self.music_queue)

        else:
            self.is_playing = False

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True







BOT.add_cog(music_cog(BOT))

with open('token.txt') as file:
    token = file.read()

keep_alive()

BOT.run(token)

    @commands.command(name='play', aliases=['p'], help='Plays a song.')
    async def play(self, ctx, *args):
        query = " ".join(args)
        if ctx.author.voice:
          voice_channel = ctx.author.voice.channel
          self.ctx = ctx
          if voice_channel is None:
              await ctx.send('Connect to a voice channel!')
          else:
              song = self.search_yt(query)
              if type(song) == type(True):
                  await ctx.send('Could not download the song. Incorrect format try another keyword. This could be due to playlist or a livestream')
              else:
                  self.music_queue.append([song, voice_channel])
                  if self.is_playing == False:
                      await self.play_music()
                  else:
                      await ctx.send(self.music_queue[len(self.music_queue)-1][0]['title']+' added to the queue')
        else:
          await ctx.send('Connect to a voice channel!')



    @commands.command(name='queue', aliases=['q'], help='List the queue songs')
    async def queue(self, ctx):
        retval = ''
        for i in range(0, len(self.music_queue)):
            retval += self.music_queue[i][0]['title'] + '\n'

        print(retval)
        if retval != '':
            await ctx.send(retval)
        else:
            await ctx.send('No music in queue')

    @commands.command(name='skip', aliases=['sk'],help='Skip the current song')
    async def skip(self, ctx):
        if self.vc != '':
              ctx.voice_client.stop()
              await ctx.send('after using skip'+ str(self.music_queue))
              await self.play_music()


    @commands.command(name='stop', aliases=['st'], help='Stop the song and clear queue songs')
    async def stop(self, ctx):
      if self.vc != '':
        ctx.voice_client.stop()
        await ctx.send('Stoped')
        self.music_queue = []

    @commands.command(name='pause', aliases=['pp'], help='Pause the current song')
    async def pause(self, ctx):
      if self.vc != '':
        self.vc.pause()
        await ctx.send('Paused')

    @commands.command(name='resume', aliases=['re'], help='Resume the paused song')
    async def resume(self, ctx):
      if self.vc != '':
        self.vc.resume()
        await ctx.send('Resumed')

    @commands.command(name='disconnect', aliases=['dc'], help='Disconnect')
    async def disconnect(self, ctx):
      if self.vc != '':
        self.music_queue = []
        self.vc = ''
        await ctx.voice_client.disconnect()
        await ctx.send('Disconnected')

    @commands.command(name='completemysentence', aliases=['cms'])
    async def completemysentence(self, ctx, name):
      if name in ['vipul', 'Vipul']:
        await ctx.send('Chutea he sala!')

      elif name in ['himanshu', 'Himanshu']:
        await ctx.send('MOTA he sala!')


    # @commands.command(name='loop', aliases=['loo'], help='Loop the current song')
    # async def loop(self, ctx):
    #   if self.loop == False:
    #     self.loop = True
    <?php
$MERCHANT_KEY = "ZHsuRZmv";
$SALT = "BL3uaRBzfK";
// Merchant Key and Salt as provided by Payu.

///$PAYU_BASE_URL = "https://sandboxsecure.payu.in";		// For Sandbox Mode
$PAYU_BASE_URL = "https://secure.payu.in";			// For Production Mode
$posted = array();
if(!empty($_POST['txnid'])) {
    //print_r($_POST);
  foreach($_POST as $key => $value) {    
    $posted[$key] = $value; 
  }
}

$formError = 0;

if(empty($posted['txnid'])) {
  // Generate random transaction id
  $txnid = substr(hash('sha256', mt_rand() . microtime()), 0, 20);
} else {
  $txnid = $posted['txnid'];
}
$hash = '';
// Hash Sequence
$hashSequence = "key|txnid|amount|productinfo|firstname|email|udf1|udf2|udf3|udf4|udf5|udf6|udf7|udf8|udf9|udf10";
if(empty($posted['hash']) && sizeof($posted) > 0) {
  if(
          empty($posted['key'])
          || empty($posted['txnid'])
          || empty($posted['amount'])
          || empty($posted['firstname'])
          || empty($posted['email'])
          || empty($posted['phone'])
          || empty($posted['productinfo'])
          || empty($posted['surl'])
          || empty($posted['furl'])
		  || empty($posted['service_provider'])
  ) {
    $formError = 1;
  } else {
    //$posted['productinfo'] = json_encode(json_decode('[{"name":"tutionfee","description":"","value":"500","isRequired":"false"},{"name":"developmentfee","description":"monthly tution fee","value":"1500","isRequired":"false"}]'));
	$hashVarsSeq = explode('|', $hashSequence);
    $hash_string = '';	
	foreach($hashVarsSeq as $hash_var) {
      $hash_string .= isset($posted[$hash_var]) ? $posted[$hash_var] : '';
      $hash_string .= '|';
    }

    $hash_string .= $SALT;

    $prod=$obj_database->selectAllData(TB_PREFIX."item","id='".$_POST['product']."'","desc","id");

    $r=rand(100,999);
	$data['bill_no']="GDI-".date('Y-m-d')."/".$r;
	$data['date']=date('Y-m-d');
	$data['user_id']=$_SESSION['user_detail']['id'];
	$data['amount']=$_SESSION['cart']['total'];
	$data['goodmart_point']=get_account_balance($_SESSION['user_detail']['user_id']);
	$wp=get_win_balance($_SESSION['user_detail']['user_id']); $lwp=($data['amount']*10/100>$wp)?$wp:$data['amount']*10/100;
	$data['win_point']=$lwp;
	$data['payable']=$data['amount']-$data['goodmart_point']-$data['win_point'];
	$data['status']=0;
	$data['shipping']=$_SESSION['shipping'];
	$data['shipping_address'] = 'Name : '.  $posted['firstname'] . '; ' . 'Phone : ' . $posted['phone']. '; ' .'Address : ' . $posted['address1'] . ', ' . $posted['city'] . ', ' . $posted['state'] . ', ' . $posted['zipcode'];
	$data['payment_mode']=1;
	
		if($obj_database->insertData($data,TB_PREFIX."invoice"))
				{
				    $bill_id=$obj_database->get_last_id();
					   foreach($_SESSION['cart']['item'] as $i) 
					   {
					       $product=$obj_database->selectAllData(TB_PREFIX."item","id='".$i."'","desc","id");
						   $data1['bill_id_fk']=$bill_id;
						   $data1['product_id']=$i;
						   $data1['user_id']=$_SESSION['user_detail']['id'];
						   $data1['quantity']=$_SESSION['cart']['quantity'][$i];
						   $data1['color']=$_SESSION['cart']['color'][$i];
						   $data1['size']=$_SESSION['cart']['size'][$i];
						   $data1['amount']=$_SESSION['cart']['quantity'][$i]*$product[0]['offer_price'];
						   $final['amount']+=$data1['amount'];
						   $data1['status']=1;
						   $obj_database->insertData($data1,TB_PREFIX."invoice_item");
					   }
					   
		    $obj_database->updateData($final,TB_PREFIX."invoice",$data['bill_no'],"bill_no"); 
			$win['user_id']=$_SESSION['user_detail']['user_id'];
		    $win['point']=$data['win_point'];
		    $win['description']="Win Point Reedeemed for Order No. ".$data['bill_no'];
		    $win['type']=2;
		    $win['status']=1;
		    $obj_database->insertData($win,TB_PREFIX."win");
		   
		   	$gdp['user_id']=$_SESSION['user_detail']['user_id'];
		    $gdp['amount']=$data['goodmart_point'];
		    $gdp['description']="Goodmart Point Reedeemed for Order No. ".$data['bill_no'];
		    $gdp['type']=2;
		    $gdp['transaction_type']=2;
		    $gdp['status']=1;
		    $obj_database->insertData($gdp,TB_PREFIX."account");
		   	}
 unset($_SESSION['cart']);
    $hash = strtolower(hash('sha512', $hash_string));
    header($PAYU_BASE_URL . '/_payment');
  }
} elseif(!empty($posted['hash'])) {
    $prod=$obj_database->selectAllData(TB_PREFIX."item","id='".$_POST['product']."'","desc","id");

    $r=rand(100,999);
	$data['bill_no']="GDI-".date('Y-m-d')."/".$r;
	$data['date']=date('Y-m-d');
	$data['user_id']=$_SESSION['user_detail']['id'];
	$data['amount']=$_SESSION['cart']['total'];
	$data['goodmart_point']=get_account_balance($_SESSION['user_detail']['user_id']);
	$wp=get_win_balance($_SESSION['user_detail']['user_id']); $lwp=($data['amount']*10/100>$wp)?$wp:$data['amount']*10/100;
	$data['win_point']=$lwp;
	$data['payable']=$data['amount']-$data['goodmart_point']-$data['win_point'];
	$data['status']=0;
	$data['shipping']=$_SESSION['shipping'];
	$data['shipping_address'] = 'Name : '.  $posted['firstname'] . '; ' . 'Phone : ' . $posted['phone']. '; ' .'Address : ' . $posted['address1'] . ', ' . $posted['city'] . ', ' . $posted['state'] . ', ' . $posted['zipcode'];
	$data['payment_mode']=1;
	
		if($obj_database->insertData($data,TB_PREFIX."invoice"))
				{
				    $bill_id=$obj_database->get_last_id();
					   foreach($_SESSION['cart']['item'] as $i) 
					   {
					       $product=$obj_database->selectAllData(TB_PREFIX."item","id='".$i."'","desc","id");
						   $data1['bill_id_fk']=$bill_id;
						   $data1['product_id']=$i;
						   $data1['user_id']=$_SESSION['user_detail']['id'];
						   $data1['quantity']=$_SESSION['cart']['quantity'][$i];
						   $data1['color']=$_SESSION['cart']['color'][$i];
						   $data1['size']=$_SESSION['cart']['size'][$i];
						   $data1['amount']=$_SESSION['cart']['quantity'][$i]*$product[0]['offer_price'];
						   $final['amount']+=$data1['amount'];
						   $data1['status']=1;
						   $obj_database->insertData($data1,TB_PREFIX."invoice_item");
					   }
					   
		    $obj_database->updateData($final,TB_PREFIX."invoice",$data['bill_no'],"bill_no"); 
			$win['user_id']=$_SESSION['user_detail']['user_id'];
		    $win['point']=$data['win_point'];
		    $win['description']="Win Point Reedeemed for Order No. ".$data['bill_no'];
		    $win['type']=2;
		    $win['status']=1;
		    $obj_database->insertData($win,TB_PREFIX."win");
		   
		   	$gdp['user_id']=$_SESSION['user_detail']['user_id'];
		    $gdp['amount']=$data['goodmart_point'];
		    $gdp['description']="Goodmart Point Reedeemed for Order No. ".$data['bill_no'];
		    $gdp['type']=2;
		    $gdp['transaction_type']=2;
		    $gdp['status']=1;
		    $obj_database->insertData($gdp,TB_PREFIX."account");
		   	}
 unset($_SESSION['cart']);
  $hash = $posted['hash'];
  header($PAYU_BASE_URL . '/_payment');
}
?>

<input type="hidden" name="key" value="<?php echo $MERCHANT_KEY ?>" />
      <input type="hidden" name="hash" value="<?php echo $hash ?>"/>
      <input type="hidden" name="txnid" value="<?php echo $txnid ?>" />
      <input type="text" name="txnid" value="<?php  echo $_POST['txnid'] . $posted['txnid'] ?>" />
        <input type="hidden" name="productinfo" value="Online Purchase" />
       <input type="hidden" name="surl" value="https://goodmart.ind.in/complete-order.php" />
       <input type="hidden" name="furl" value="https://goodmart.ind.in/payment-failed.php" />
       <input type="hidden" name="service_provider" value="payu_paisa" size="64" />







BOT.add_cog(music_cog(BOT))


# keep_alive()

BOT.run('ODcwMTE4NzM3NDUyMzQzMzE3.YQIHOQ.NVAr2U5FdU8iR11KWyGT_x3YwRg')
