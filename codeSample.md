### Charlie Wright Code Examples and Commentary

```javascript
define(["jQueryFramework", "./highlighterSerializer"], function ($, highlighterSerializer) {
	"use strict";
	
	function initializeCanvasHighlighter(value, $root, width, height) {
			var $canvas = $root.find('canvas');
			var $clear = $root.find(".clear").click(clear);
			var $highlight = $root.find(".highlight").click(highlight);
			var $erase = $root.find(".erase").click(erase);
			var $cursor = $root.find(".highlighter-cursor");
			var canvas = $canvas[0];
			var drawing = false;
			var lastPos = null;
			var ctx = canvas.getContext("2d");

			function moveCursor(e) {
				$cursor.css({
					left: e.offsetX,
					top: e.offsetY
				});
			}

			function clear() {
				ctx.clearRect(0, 0, canvas.width, canvas.height);
				value.set("");
			}

			function highlight() {
				ctx.globalCompositeOperation = "source-over";
				$highlight.addClass("set");
				$erase.removeClass("set");
				$cursor.removeClass("erase");
			}

			function erase() {
				ctx.globalCompositeOperation = "destination-out";
				$erase.addClass("set");
				$highlight.removeClass("set");
				$cursor.addClass("erase");
			}

			function showCursor() {
				$cursor.removeClass("hidden");
			}

			function hideCursor() {
				$cursor.addClass("hidden");
			}

			function startDrawing(e) {
				var pos = getCursorPosition(e);
				if (!drawing) { //first stroke, draw a dot
					ctx.beginPath();
					ctx.moveTo(pos.x, pos.y);
					ctx.lineTo(pos.x, pos.y);
					ctx.stroke();
				}
				drawing = true;
				lastPos = pos;
				//hijack the event and prevent default so that we don"t select the text on the page
				e.preventDefault();
			}

			function stopDrawing(e) {
				if (drawing) {
					drawing = false;
					saveResponse(canvas);
				}
			}

			function drawTo(e) {
				if (drawing) {
					var pos = getCursorPosition(e);
					ctx.beginPath();
					ctx.moveTo(lastPos.x, lastPos.y);
					ctx.lineTo(pos.x, pos.y);
					ctx.stroke();
					lastPos = pos;
				}
			}

			function getCursorPosition(e) {
				if (e.clientX) {
					return {
						x: e.clientX - canvas.getBoundingClientRect().left,
						y: e.clientY - canvas.getBoundingClientRect().top
					};
				} else //touch event
				{
					return {
						x: e.originalEvent.touches[0].clientX - canvas.getBoundingClientRect().left,
						y: e.originalEvent.touches[0].clientY - canvas.getBoundingClientRect().top
					};
				}

			}

			function saveResponse(canvas) {
				var empty = document.createElement('canvas');
				empty.width = canvas.width;
				empty.height = canvas.height;

				if (canvas.toDataURL() == empty.toDataURL()) {
					value.set("");
					return;
				}
				var encodedResult = btoa(highlighterSerializer.byteArrayToDecoded(highlighterSerializer.getByteArrayFromCanvasObject(canvas)));
				value.set("[0,0," + canvas.width + "," + canvas.height + "]" + encodedResult);
			}

			canvas.width = width;
			canvas.height = height;
			ctx.lineCap = "round";
			ctx.lineJoin = "round";
			ctx.lineWidth = 10;
			ctx.strokeStyle = "rgba(255, 255, 0, 1)"; //highlighter is yellow by default

			$canvas.mousedown(function (e) {
				if(e.which == 1) //left click
					startDrawing(e);
			});
			$canvas.mouseup(stopDrawing);
			$canvas.mouseout(stopDrawing);
			$canvas.mousemove(drawTo);
			$canvas.on("touchstart", startDrawing);
			$canvas.on("touchmove", drawTo);
			$canvas.on("touchend", stopDrawing);

			$canvas.bind("mouseenter", showCursor);
			$canvas.bind("mousemove", moveCursor);
			$canvas.bind("mouseleave", hideCursor);

			if (value.get()) {
				highlighterSerializer.drawPixelsOnCanvas(canvas, value.get());
			}
		}

	return {
		init: initializeCanvasHighlighter
	};
});

```