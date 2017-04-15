define([], function () {
	"use strict";

	function decodedToByteArray(decodedString) {
		var byteArr = [];
		for (var i = 0; i < decodedString.length; i++) {
			byteArr.push(decodedString.charCodeAt(i));
		}
		return byteArr;
	}

	function byteArrayToDecoded(byteArr) {
		var decoded = "";
		for (var i = 0; i < byteArr.length; i++) {
			decoded += String.fromCharCode(byteArr[i]);
		}
		return decoded;
	}

	function getByteArrayFromCanvasObject(canvas) {
		// iterate through every pixel. If it is a highlighted pixel, then stuff something into the byte array at that index
		// this thing is going to start at 0, 0, and have the height and width!
		var canvasImageData = canvas.getContext("2d").getImageData(0, 0, canvas.width, canvas.height);
		var bytes = [];
		var bits = canvasImageData.data;
		var masks = [128, 64, 32, 16, 8, 4, 2, 1];
		for (var i = 0; i < bits.length; i += 32) {
			var nextByte = 0;
			for (var j = 0; j < masks.length; j++) {
				if (bits[i + (j * 4)]) {
					nextByte += masks[j];
				}
			}
			bytes.push(nextByte);
		}
		return bytes;

	}

	function drawPixelsOnCanvas(canvas, highlighterData) {
		var encoded = highlighterData.substr(highlighterData.indexOf("]") + 1);
		var decoded = atob(encoded); //the charcodeat each index is the byte value of the array
		var byteArr = decodedToByteArray(decoded);
		//This code is adapted from the SNG heatmap using sparq data
		var compareBits = [128, 64, 32, 16, 8, 4, 2, 1];
		var context = canvas.getContext("2d");

		var imgData = context.createImageData(canvas.width, canvas.height);

		for (var i = 0; i < byteArr.length; i++) {
			for (var j = 0; j < 8; j++) {
				if ((byteArr[i] & compareBits[j]) == compareBits[j]) {
					var index = (i * 8 + j) * 4;
					imgData.data[index] = 255;
					imgData.data[index + 1] = 255;
					imgData.data[index + 2] = 0;
					imgData.data[index + 3] = 255;
				}
			}
		}
		context.putImageData(imgData, 0,0);
	}

	return {
		drawPixelsOnCanvas: drawPixelsOnCanvas,
		getByteArrayFromCanvasObject: getByteArrayFromCanvasObject,
		decodedToByteArray: decodedToByteArray,
		byteArrayToDecoded: byteArrayToDecoded
	};
});