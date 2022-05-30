<script>
	import P5 from 'p5-svelte';
	import 'papercss/dist/paper.min.css'
	import Slider from "spaper/components/Form/Slider.svelte";
	import Form from "spaper/components/Form/Form.svelte";

	const xs = []; 
	const ys = []; 

	$: optimizer = tf.train.sgd(lr) || lr; 

	let m, b; // the coefficients
	let lr = 0.01; 
	let optimizer = tf.train.sgd(lr);
	let epochs = 200; // hmm we could create like a slider for this 

	const loss_fn = (pred, labels) => {
		return pred.sub(labels).square().mean();
	} // MSE

	function predict(x) {
		return tf.tensor1d(x).mul(m).add(b);
	}
	 
	const sketch = (p5) => {
		p5.setup = () => {
			p5.createCanvas(400, 400);

			m = tf.variable(tf.scalar(p5.random(1)));
			b = tf.variable(tf.scalar(p5.random(1))); // initialize to random values
			
		};	

		p5.mousePressed = () => {

			if ((p5.mouseX < 0 || p5.mouseY < 0) || (p5.mouseX > p5.width || p5.mouseY > p5.height)) {
				return;
			}

			const x = p5.map(p5.mouseX, 0, p5.width, 0, 1); // convert to the graph coordinates
			const y = p5.map(p5.mouseY, 0, p5.height, 1, 0); // go from 1 to 0 as higher up (lower value) means higher y-coordinate
			xs.push(x);
			ys.push(y);

			// after placing the points, you want to to optimize the model right
			for (let i =0; i < epochs; i++) {
				if (xs.length == 0) {
					break;
				}

				optimizer.minimize(() => loss_fn(predict(xs), tf.tensor1d(ys)));
			}
		};

		p5.draw = () => {
			p5.background(0);
			p5.stroke(255);
			p5.strokeWeight(12);

			for (let i = 0; i < xs.length; i++) {
				let px = p5.map(xs[i], 0, 1, 0, p5.width); // reconvert back to the page range
				let py = p5.map(ys[i], 0, 1, p5.height, 0); // higher value should be closer to height
				p5.point(px, py);
			}

			p5.stroke(255);
			p5.strokeWeight(4);
			const preds = predict([0, 1]); 
			const y1 = p5.map(preds.dataSync()[0], 0, 1, p5.height, 0);
			const y2 = p5.map(preds.dataSync()[1], 0, 1, p5.height, 0);

			p5.line(0, y1, p5.width, y2);
		
		}
	}
</script>

<link rel="stylesheet" href="https://unpkg.com/papercss@1.8.3/dist/paper.min.css">

<div class="center"> 
	<h1> Linear Regression Demo to Piss Off Ronak </h1>
	<P5 {sketch} />

	<br> 
	<!-- add the sliders for the lr and epochs --> 	
	<Form class="sliders"> 
		<Slider label="learning rate" min="0" max="1" step="0.01" bind:value={lr}></Slider>
		<br>
		<br> 
		<Slider label="epochs" min="1" max="200" step="1" bind:value={epochs}></Slider>
	</Form>

	<p style="font-size: 2em;"> Basically ronak doesn't like p5.js. i used papercss tho so i dont think he'll be too mad </p>
	<footer> this is basically a small addition of the coding train challenge <a href="https://www.youtube.com/watch?v=dLp10CFIvxI"> in this video </a>  </footer>
</div>

<style> 
	a {
		text-decoration: none;
	}

	.center {
		text-align: center;
	}

	.sliders {
		display: inline-block;
		width: 50%; 
	}
</style> 
