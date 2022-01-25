<script lang="ts">
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let _div: d3.Selection<d3.BaseType, unknown, HTMLElement, any>;
  let div = _div as unknown | SVGElement;
  let reader: FileReader;
  let file: File;

  onMount(() => {
    // Check for the various File API support.
    if (window.File && window.FileReader && window.FileList && window.Blob) {
      /* alert(" Great success! All the File APIs are supported."); */
    } else {
      alert("The File APIs are not fully supported in this browser.");
    }

    _div = d3.select("div");
    let input = _div.append("input");
    input
      .attr("type", "file")
      .attr("id", "file5")
      .attr("name", "file5")
      .on("change", function (event) {
        handleFileSelect(event);
      });

    let button = _div.append("button");
    button.text("読み取りをキャンセル").on("click", function (evt) {
      if (reader !== undefined) {
        reader.abort();
      } else {
        /* blank intentionall */
      }
    });

    let _progbar = _div.append("div");
    _progbar.attr("id", "progress_bar").attr("class", "");

    let _progress = _progbar.append("div");
    _progress
      .attr("class", "percent")
      .attr("style", "width : 100%;")
      .text("0%");

    let p = _div.append("p");
    let strng = p.append("strong");
    strng.text("ヒント");
    let span = p.append("span");
    span.text(
      ":この進行状況インジケータを実際に表示するには、サイズの大きいファイルかリモートドライブ上のリソースを試して下さい。"
    );
  });

  function handleFileSelect(event: Event) {
    let prgs = _div.selectChild("#progress_bar").selectChild(".percent");
    prgs.attr("style", "width : 0%");
    prgs.text("0%");

    reader = new FileReader();

    reader.onerror = function (e) {
      errorhandler(e);
    };

    reader.onprogress = function (e) {
      updateProgress(e);
    };

    reader.onabort = function (e) {
      alert("File read cancelled");
    };
    reader.onloadstart = function (e) {
      let prgbar = _div.selectChild("#progress_bar");
      prgbar.attr("class", "loading");
    };
    reader.onload = function (e) {
      // Ensure that the progress bar displays 100% at the end.
      let prgs = _div.selectChild("#progress_bar").selectChild(".percent");
      prgs.attr("style", "width : 100%");
      prgs.text("100%");

      let prgbar = _div.selectChild("#progress_bar");
      setTimeout(() => prgbar.attr("class", ""), 2000);
    };

    file = Array.from((event.target as HTMLInputElement).files)[0];
    reader.readAsBinaryString(file);
  }

  function errorhandler(event: ProgressEvent) {
    switch ((event.target as FileReader).error.code) {
      case (event.target as FileReader).error.NOT_FOUND_ERR:
        alert("File Not Found!");
        break;
      case (event.target as FileReader).error.NOT_SUPPORTED_ERR:
        alert("This File Not Supported!");
        break;
      default:
        alert("An error occurred reading this file.");
    }
  }

  function updateProgress(event: ProgressEvent) {
    // evt is a ProgressEvent.
    if (event.lengthComputable) {
      var percentLoaded = Math.round((event.loaded / event.total) * 100);
      // Increase the progress bar length.
      if (percentLoaded < 100) {
        let prgs = _div.selectChild("#progress_bar").selectChild(".percent");
        prgs.attr("style", "width : " + percentLoaded + "%");
        prgs.text(percentLoaded + "%");
      }
    }
  }
</script>

<div bind:this={div} class="example" />

<style lang="scss">
  .example {
    padding: 10px;
    border: 1px solid #ccc;
    :global(button) {
      background: #333;
      color: #fff;
      border-radius: 4px;
      border: 0;
      padding: 10px 14px;
    }
    :global(#progress_bar) {
      margin: 10px 0;
      padding: 3px;
      border: 1px solid #000;
      font-size: 14px;
      clear: both;
      opacity: 0;
      transition: opacity 1s linear;
      :global(.percent) {
        background-color: #99ccff;
        height: auto;
        width: 0;
      }
    }
    :global(#progress_bar.loading) {
      opacity: 1;
    }
  }
</style>
