<script lang="ts">
  import { FFmpeg } from "@ffmpeg/ffmpeg";
  import { fetchFile, toBlobURL } from "@ffmpeg/util";
  import { onMount } from "svelte";
  let videoUrl: string  = "https://raw.githubusercontent.com/ffmpegwasm/testdata/master/Big_Buck_Bunny_180_10s.webm"
  let ffmpeg;
  let message = '';
  let duration = null;
  let video: HTMLVideoElement | null = null;
    let videoProgress : number = 0
  onMount(() => {
    video = document.createElement('video');
    video.src = videoUrl;

    video.onloadedmetadata = () => {
      duration = video.duration;
      console.log(duration)
    };
  });

  const load = async () => {
        const baseURL = 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/esm'
         ffmpeg = new FFmpeg();
         ffmpeg.on('log', ({ message }) => {
      message = message;
      const regexResult = /time=([0-9:.]+)/.exec(message);
      if (regexResult && regexResult?.[1]) {
        const howMuchIsDone : string = regexResult?.[1];
        const [hours,minutes,seconds] : string[]= howMuchIsDone.split(':');
        const doneTotalSeconds = Number(hours) * 3600 + Number(minutes) * 60 + Number(seconds);
         videoProgress = doneTotalSeconds / duration;
        console.log(videoProgress)
    }

    });
        // toBlobURL is used to bypass CORS issue, urls with the same
        // domain can be used directly.
        await ffmpeg.load({
            coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript'),
            wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
        });
        console.log("loaded")
        
	};

  const transcode = async () => {
   /* const baseURL = 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/esm'
         ffmpeg = new FFmpeg();
        
        // toBlobURL is used to bypass CORS issue, urls with the same
        // domain can be used directly.
        await ffmpeg.load({
            coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript'),
            wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
        });

        */
    await ffmpeg.writeFile(
      "input.webm",
      await fetchFile(
        "https://raw.githubusercontent.com/ffmpegwasm/testdata/master/Big_Buck_Bunny_180_10s.webm"
      )
    );
    await ffmpeg.writeFile(
      "arial.ttf",
      await fetchFile(
        "https://raw.githubusercontent.com/ffmpegwasm/testdata/master/arial.ttf"
      )
    );
    await ffmpeg.exec([
      "-i",
      "input.webm",
      "-vf",
      "drawtext=fontfile=/arial.ttf:text='ffmpeg.wasm':x=10:y=10:fontsize=24:fontcolor=white",
      "output.mp4",
    ]);
    const data = await ffmpeg.readFile("output.mp4");
    console.log(data)
    videoUrl = URL.createObjectURL(new Blob([data.buffer], {type: 'video/mp4'}));
  console.log(videoUrl)
  };
</script>
<button on:click={load}> Load</button>
<!-- svelte-ignore a11y-media-has-caption -->
<video src={videoUrl} controls ></video><br/>

<button on:click={transcode}>Add Sub</button>
<p>{ Math.ceil(videoProgress * 100 + 0.1) }%</p>