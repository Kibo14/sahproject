<script>
// @ts-nocheck

  import { onMount, onDestroy } from 'svelte';
  import { Chessground } from 'chessground';
  import { Chess } from 'chess.js';
  
  import 'chessground/assets/chessground.base.css';
  import 'chessground/assets/chessground.brown.css';
  import 'chessground/assets/chessground.cburnett.css';

  // Props
  export let fen = 'rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1'; // Default starting position
  export let orientation = 'white';
  export let movable = true;
  
  // Internal state
  let chessground;
  let element;
  let chess = new Chess(fen);
  
  // Initialize Chessground when component mounts
  onMount(() => {
    const config = {
      fen: fen,
      orientation: orientation,
      movable: {
        free: false,
        color: movable ? 'both' : null,
        dests: toDests(chess),
        events: {
          after: handleMove
        }
      }
    };
    
    chessground = Chessground(element, config);
    
    return () => {
      if (chessground) {
        chessground.destroy();
      }
    };
  });
  
  // Handle move logic
  function handleMove(orig, dest) {
    const move = chess.move({
      from: orig,
      to: dest,
      promotion: 'q' // Always promote to queen for simplicity
    });
    
    if (move) {
      // Update the board after the move
      chessground.set({
        fen: chess.fen(),
        turnColor: chess.turn() === 'w' ? 'white' : 'black',
        movable: {
          color: chess.turn() === 'w' ? 'white' : 'black',
          dests: toDests(chess)
        }
      });
    }
  }
  
  // Convert chess.js moves to Chessground destinations format
  function toDests(chess) {
    const dests = new Map();
    const moves = chess.moves({ verbose: true });
    
    moves.forEach(move => {
      if (dests.has(move.from)) {
        dests.get(move.from).push(move.to);
      } else {
        dests.set(move.from, [move.to]);
      }
    });
    
    return dests;
  }
</script>

<div class="chessboard" bind:this={element}></div>

<style>
  .chessboard {
    width: 600px;
    height: 600px;
    margin: 0 auto;
  }
</style>