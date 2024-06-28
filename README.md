Importações e Configurações Iniciais
python
Copiar código
import discord
from discord.ext import commands
import yt_dlp as youtube_dl
import subprocess
from collections import deque
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials
import re
Aqui, estamos importando todas as bibliotecas necessárias para o funcionamento do bot.

discord e commands são utilizados para interagir com o Discord.
yt_dlp é uma biblioteca para baixar e extrair informações de vídeos do YouTube.
subprocess permite executar comandos do sistema, usado aqui para verificar o ffmpeg.
deque da coleção collections é utilizado para gerenciar a fila de músicas.
spotipy e SpotifyClientCredentials são usados para acessar a API do Spotify.
re é a biblioteca de expressões regulares para validar URLs.
Configuração de Intenções do Bot
python
Copiar código
intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)
Aqui, configuramos as intenções do bot, permitindo que ele leia o conteúdo das mensagens, e criamos uma instância do bot com o prefixo de comando !.

Fila de Músicas
python
Copiar código
music_queue = deque()
Criamos uma fila (deque) para armazenar as músicas a serem reproduzidas.

Configurações do Spotify
python
Copiar código
SPOTIPY_CLIENT_ID = 'YOUR_SPOTIPY_CLIENT_ID'
SPOTIPY_CLIENT_SECRET = 'YOUR_SPOTIPY_CLIENT_SECRET'
sp = spotipy.Spotify(auth_manager=SpotifyClientCredentials(client_id=SPOTIPY_CLIENT_ID, client_secret=SPOTIPY_CLIENT_SECRET))
Aqui, configuramos as credenciais do Spotify para acessar a API e criar uma instância do cliente Spotify.

Evento on_ready
python
Copiar código
@bot.event
async def on_ready():
    print(f'Bot {bot.user} está online!')
Este evento é chamado quando o bot está pronto e conectado ao Discord. Ele imprime uma mensagem no console.

Comando join
python
Copiar código
@bot.command(name='join')
async def join(ctx):
    if ctx.author.voice:
        channel = ctx.author.voice.channel
        await channel.connect()
    else:
        await ctx.send("Você não está em um canal de voz!")
Este comando faz o bot se juntar ao canal de voz em que o usuário está.

Comando leave
python
Copiar código
@bot.command(name='leave')
async def leave(ctx):
    if ctx.voice_client:
        await ctx.voice_client.disconnect()
    else:
        await ctx.send("Não estou em um canal de voz!")
Este comando faz o bot sair do canal de voz.

Comando play
python
Copiar código
@bot.command(name='play')
async def play(ctx, *, query):
    if ctx.voice_client:
        try:
            # Verificar se a query é uma URL do YouTube
            if re.match(r'https?://(www\.)?(youtube\.com|youtu\.be)/.+', query):
                # URL do YouTube
                music_queue.append(query)
                await ctx.send(f"Adicionado URL do YouTube à fila.")
            # Verificar se a query é uma URL de playlist do Spotify
            elif re.match(r'https?://(open\.)?spotify\.com/playlist/.*', query):
                playlist_id = query.split('/')[-1].split('?')[0]
                results = sp.playlist_tracks(playlist_id)
                ctx.bot.last_playlist = results  # Armazenar a última playlist carregada
                for item in results['items']:
                    track = item['track']
                    track_name = track['name']
                    artist_name = track['artists'][0]['name']
                    music_queue.append(f"ytsearch:{track_name} {artist_name}")
                await ctx.send(f"Adicionado {len(results['items'])} músicas da playlist à fila.")
            # Verificar se a query é uma URL de música do Spotify
            elif re.match(r'https?://(open\.)?spotify\.com/track/.*', query):
                track_id = query.split('/')[-1].split('?')[0]
                track = sp.track(track_id)
                track_name = track['name']
                artist_name = track['artists'][0]['name']
                music_queue.append(f"ytsearch:{track_name} {artist_name}")
                await ctx.send(f"Adicionada a música {track_name} à fila.")
            else:
                # Pesquisa por nome
                results = sp.search(q=query, limit=1)
                if not results['tracks']['items']:
                    await ctx.send("Nenhuma música encontrada com esse nome!")
                    return

                track = results['tracks']['items'][0]
                track_name = track['name']
                artist_name = track['artists'][0]['name']
                music_queue.append(f"ytsearch:{track_name} {artist_name}")
                await ctx.send(f"Adicionada a música {track_name} à fila.")

            if not ctx.voice_client.is_playing():
                await play_next(ctx)
        except Exception as e:
            await ctx.send(f"Ocorreu um erro ao tentar reproduzir a música: {str(e)}")
            print(f"Erro: {str(e)}")
    else:
        await ctx.send("Não estou em um canal de voz!")
Este comando adiciona músicas à fila de reprodução. Ele verifica se a entrada é uma URL do YouTube, uma URL de playlist do Spotify, uma URL de música do Spotify ou uma pesquisa por nome. Dependendo do tipo de entrada, ele adiciona a música ou as músicas correspondentes à fila e, se nada estiver tocando, chama a função play_next.

Função play_next
python
Copiar código
async def play_next(ctx):
    if len(music_queue) > 0:
        query = music_queue.popleft()
        ydl_opts = {
            'format': 'bestaudio/best',
            'noplaylist': True,
            'default_search': 'ytsearch',
            'quiet': True
        }
        with youtube_dl.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(query, download=False)
            url2 = info['entries'][0]['url'] if 'entries' in info else info['url']
        
        ffmpeg_path = "C:\\Program Files\\ffmpeg\\bin\\ffmpeg.exe"  # Especifique o caminho completo para o ffmpeg

        # Log the URL and ffmpeg path for debugging
        print(f"URL: {url2}")
        print(f"FFmpeg Path: {ffmpeg_path}")

        # Ensure ffmpeg is working by running a test command
        try:
            subprocess.check_output([ffmpeg_path, "-version"])
            print("FFmpeg is working correctly.")
        except subprocess.CalledProcessError as e:
            print(f"Error with FFmpeg: {e.output}")
            await ctx.send("Erro ao verificar o FFmpeg.")
            return

        # Create FFmpeg audio source with proper parameters
        source = discord.FFmpegPCMAudio(url2, executable=ffmpeg_path, before_options="-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5", options="-vn")
        ctx.voice_client.play(source, after=lambda e: bot.loop.create_task(play_next(ctx)))
    else:
        await ctx.send("A fila de músicas está vazia! Saindo do canal de voz.")
        #await ctx.voice_client.disconnect()
Esta função é chamada para reproduzir a próxima música na fila. Ela utiliza yt_dlp para extrair informações sobre a música e cria uma fonte de áudio com FFmpegPCMAudio, que é então reproduzida. Se não houver mais músicas na fila, o bot envia uma mensagem avisando que a fila está vazia.

Comandos de Controle de Reprodução
Pause
python
Copiar código
@bot.command(name='pause')
async def pause(ctx):
    if ctx.voice_client and ctx.voice_client.is_playing():
        ctx.voice_client.pause()
    else:
        await ctx.send("Nada está tocando no momento!")
Resume
python
Copiar código
@bot.command(name='resume')
async def resume(ctx):
    if ctx.voice_client and ctx.voice_client.is_paused():
        ctx.voice_client.resume()
    else:
        await ctx.send("A música não está pausada!")
Stop
python
Copiar código
@bot.command(name='stop')
async def stop(ctx):
    if ctx.voice_client and ctx.voice_client.is_playing():
        ctx.voice_client.stop()
    else:
        await ctx.send("Nada está tocando no momento!")
Skip
python
Copiar código
@bot.command(name='skip')
async def skip(ctx):
    if ctx.voice_client and ctx.voice_client.is_playing():
        ctx.voice_client.stop()
        await play_next(ctx)
    else:
        await ctx.send("Nada está tocando no momento!")
Esses comandos permitem pausar, retomar, parar e pular músicas na fila de reprodução.

Comando swap
python
Copiar código
@bot.command(name='swap')
async def swap(ctx, query: str):
    try:
        # Verificar se a query é um número
        if query.isdigit():
            track_number = int(query)
            # Certifique-se de que uma playlist foi carregada anteriormente
            if 'last_playlist' in ctx.bot.__dict__:
                results = ctx.bot.last_playlist
                if 1 <= track_number <= len(results['items']):
                    chosen_track = results['items'][track_number - 1]['track
']
track_name = chosen_track['name']
artist_name = chosen_track['artists'][0]['name']
music_queue.appendleft(f"ytsearch:{track_name} {artist_name}")

python
Copiar código
                if ctx.voice_client.is_playing():
                    ctx.voice_client.stop()

                await play_next(ctx)
                await ctx.send(f"Tocando agora: {track_name} - {artist_name}")
            else:
                await ctx.send("Número da música inválido!")
        else:
            await ctx.send("Nenhuma playlist carregada. Por favor, adicione uma playlist primeiro.")
    # Verificar se a query é uma URL do YouTube
    elif re.match(r'https?://(www\.)?(youtube\.com|youtu\.be)/.+', query):
        # URL do YouTube
        music_queue.appendleft(query)
        if ctx.voice_client.is_playing():
            ctx.voice_client.stop()
        await play_next(ctx)
        await ctx.send(f"Adicionado URL do YouTube à fila e tocando agora.")
    # Verificar se a query é uma URL de playlist do Spotify
    elif re.match(r'https?://(open\.)?spotify\.com/playlist/.*', query):
        playlist_id = query.split('/')[-1].split('?')[0]
        results = sp.playlist_tracks(playlist_id)
        ctx.bot.last_playlist = results  # Armazenar a última playlist carregada
        await ctx.send(f"Playlist carregada com {len(results['items'])} músicas. Use !swap <número> para trocar para uma música específica.")
    else:
        await ctx.send("Por favor, forneça uma URL de playlist válida do Spotify ou um número de música.")
except Exception as e:
    await ctx.send(f"Ocorreu um erro ao tentar processar sua escolha: {str(e)}")
    print(f"Erro: {str(e)}")
arduino
Copiar código

Este comando permite trocar a música atual por outra da fila, de uma URL do YouTube ou de uma playlist do Spotify.

### Execução do Bot

```python
bot.run('YOUR_BOT_TOKEN')
Finalmente, o bot é executado com o token fornecido.

Resumo
Este bot de música para Discord usa APIs do YouTube e Spotify para adicionar músicas a uma fila e reproduzi-las em um canal de voz. Ele inclui comandos para controle de reprodução, como pausar, retomar, parar, pular e trocar músicas.
