<script lang="ts">
  import { FFmpeg } from "@ffmpeg/ffmpeg";
  import { fetchFile, toBlobURL } from "@ffmpeg/util";
  import { onMount } from "svelte";
  let videoValue ;
  let ffmpeg;
  let message = '';
  const load = async () => {
        const baseURL = 'https://unpkg.com/@ffmpeg/core@0.12.4/dist/esm'
         ffmpeg = new FFmpeg();
         ffmpeg.on('log', ({ message }) => {
      message = message;
      console.log(message);
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
    videoValue = URL.createObjectURL(new Blob([data.buffer], {type: 'video/mp4'}));
  console.log(videoValue)
  };
</script>
<button on:click={load}> Load</button>
<!-- svelte-ignore a11y-media-has-caption -->
<video src={videoValue} controls></video><br/>
<button on:click={transcode}>Add Sub</button>
