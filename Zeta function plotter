import React, { useState, useEffect, useRef, useCallback } from 'react';
import { ZoomIn, ZoomOut, RefreshCw, Calculator, MousePointer2, Move } from 'lucide-react';

const App = () => {
  // --- State ---
  const [params, setParams] = useState({ a: 0.5, b: 14.13 }); // Default to first zero
  const [nTerms, setNTerms] = useState(100);
  const [mode, setMode] = useState('zeta'); // 'zeta' or 'eta'
  const [showCorrection, setShowCorrection] = useState(true);
  
  // Viewport State
  const [viewLeft, setViewLeft] = useState({ x: 0.5, y: 10, scale: 40 });
  const [viewRight, setViewRight] = useState({ x: 0, y: 0, scale: 100 });

  // Refs for Canvas and Interaction
  const canvasLeftRef = useRef(null);
  const canvasRightRef = useRef(null);
  const leftContainerRef = useRef(null);
  const rightContainerRef = useRef(null);
  
  // Interaction State Ref (to avoid closure staleness in event handlers)
  // Stores the state of the current gesture (start points, initial view, mode)
  const interactionRef = useRef({
    left: { mode: 'none', startArgs: null, startView: null }, // mode: 'drag-point', 'pan', 'pinch'
    right: { mode: 'none', startArgs: null, startView: null }
  });

  // --- Math Helpers ---
  const complexAdd = (c1, c2) => ({ x: c1.x + c2.x, y: c1.y + c2.y });
  
  const complexDiv = (c1, c2) => {
    const denom = c2.x * c2.x + c2.y * c2.y;
    return {
      x: (c1.x * c2.x + c1.y * c2.y) / denom,
      y: (c1.y * c2.x - c1.x * c2.y) / denom
    };
  };

  const complexPow = (base, exponent) => {
    const lnBase = Math.log(base);
    const realPart = exponent.x * lnBase;
    const imagPart = exponent.y * lnBase;
    const mag = Math.exp(realPart);
    return {
      x: mag * Math.cos(imagPart),
      y: mag * Math.sin(imagPart)
    };
  };

  // Screen/World Transforms
  const screenToWorld = (sx, sy, width, height, view) => {
    const wx = (sx - width / 2) / view.scale + view.x;
    const wy = -(sy - height / 2) / view.scale + view.y; 
    return { x: wx, y: wy };
  };

  const worldToScreen = (wx, wy, width, height, view) => {
    const sx = (wx - view.x) * view.scale + width / 2;
    const sy = -(wy - view.y) * view.scale + height / 2;
    return { x: sx, y: sy };
  };

  // --- Core Math ---
  const calculateTerm = (n, a, b, useEta) => {
    const lnN = Math.log(n);
    let mag = Math.exp(-a * lnN);
    const theta = -b * lnN; 
    if (useEta && (n % 2 === 0)) mag = -mag; 
    return { x: mag * Math.cos(theta), y: mag * Math.sin(theta) };
  };

  const calculateCorrection = (N, a, b) => {
    const s = { x: a, y: b };
    const oneMinusS = { x: 1 - a, y: -b };
    const sMinusOne = { x: a - 1, y: b };
    const num = complexPow(N, oneMinusS);
    const term1 = complexDiv(num, sMinusOne);
    const termN = calculateTerm(N, a, b, false);
    const term2 = { x: -0.5 * termN.x, y: -0.5 * termN.y };
    return complexAdd(term1, term2);
  };

  // --- Rendering ---
  const drawLeft = useCallback(() => {
    const canvas = canvasLeftRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const { width, height } = canvas;
    const { x: cx, y: cy, scale } = viewLeft;

    ctx.fillStyle = '#0f172a';
    ctx.fillRect(0, 0, width, height);

    // Grid
    ctx.lineWidth = 1;
    ctx.strokeStyle = '#334155';
    const startX = Math.floor((cx - width / 2 / scale));
    const endX = Math.ceil((cx + width / 2 / scale));
    for (let i = startX; i <= endX; i++) {
      const sx = worldToScreen(i, 0, width, height, viewLeft).x;
      ctx.beginPath(); ctx.moveTo(sx, 0); ctx.lineTo(sx, height); ctx.stroke();
    }
    const startY = Math.floor((cy - height / 2 / scale));
    const endY = Math.ceil((cy + height / 2 / scale));
    for (let i = startY; i <= endY; i++) {
      const sy = worldToScreen(0, i, width, height, viewLeft).y;
      ctx.beginPath(); ctx.moveTo(0, sy); ctx.lineTo(width, sy); ctx.stroke();
    }

    // Axes
    const origin = worldToScreen(0, 0, width, height, viewLeft);
    ctx.lineWidth = 2;
    ctx.strokeStyle = '#94a3b8';
    ctx.beginPath(); ctx.moveTo(0, origin.y); ctx.lineTo(width, origin.y); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(origin.x, 0); ctx.lineTo(origin.x, height); ctx.stroke();

    // Critical Strip
    const stripLeft = worldToScreen(0, 0, width, height, viewLeft).x;
    const stripRight = worldToScreen(1, 0, width, height, viewLeft).x;
    ctx.fillStyle = 'rgba(255, 255, 0, 0.05)';
    ctx.fillRect(stripLeft, 0, stripRight - stripLeft, height);

    // Critical Line
    const criticalLineX = worldToScreen(0.5, 0, width, height, viewLeft).x;
    ctx.strokeStyle = 'rgba(255, 255, 0, 0.3)';
    ctx.setLineDash([5, 5]);
    ctx.beginPath(); ctx.moveTo(criticalLineX, 0); ctx.lineTo(criticalLineX, height); ctx.stroke();
    ctx.setLineDash([]);

    // Point s
    const point = worldToScreen(params.a, params.b, width, height, viewLeft);
    ctx.beginPath(); ctx.arc(point.x, point.y, 8, 0, Math.PI * 2);
    ctx.fillStyle = '#3b82f6'; ctx.fill();
    ctx.beginPath(); ctx.arc(point.x, point.y, 3, 0, Math.PI * 2);
    ctx.fillStyle = '#ffffff'; ctx.fill();

    ctx.fillStyle = '#e2e8f0';
    ctx.font = '12px monospace';
    ctx.fillText(`s = ${params.a.toFixed(2)} + ${params.b.toFixed(2)}i`, point.x + 12, point.y - 12);

  }, [params, viewLeft]);


  const drawRight = useCallback(() => {
    const canvas = canvasRightRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const { width, height } = canvas;
    
    ctx.fillStyle = '#0f172a';
    ctx.fillRect(0, 0, width, height);

    const origin = worldToScreen(0, 0, width, height, viewRight);
    
    ctx.lineWidth = 1;
    ctx.strokeStyle = '#334155';
    ctx.beginPath(); ctx.moveTo(0, origin.y); ctx.lineTo(width, origin.y); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(origin.x, 0); ctx.lineTo(origin.x, height); ctx.stroke();

    let curX = 0;
    let curY = 0;
    
    ctx.lineWidth = 2;
    ctx.lineJoin = 'round';
    ctx.lineCap = 'round';
    ctx.beginPath();
    ctx.moveTo(origin.x, origin.y);

    for (let n = 1; n <= nTerms; n++) {
      const term = calculateTerm(n, params.a, params.b, mode === 'eta');
      curX += term.x;
      curY += term.y;
      const screenPos = worldToScreen(curX, curY, width, height, viewRight);
      ctx.lineTo(screenPos.x, screenPos.y);
    }
    
    ctx.strokeStyle = mode === 'eta' ? '#10b981' : '#06b6d4';
    ctx.stroke();

    const sumScreen = worldToScreen(curX, curY, width, height, viewRight);
    ctx.fillStyle = '#ef4444';
    ctx.beginPath(); ctx.arc(sumScreen.x, sumScreen.y, 4, 0, Math.PI * 2); ctx.fill();

    let trueValue = { x: curX, y: curY };

    if (showCorrection && mode === 'zeta') {
        const correction = calculateCorrection(nTerms, params.a, params.b);
        trueValue = complexAdd({ x: curX, y: curY }, correction);
        const trueScreen = worldToScreen(trueValue.x, trueValue.y, width, height, viewRight);

        ctx.strokeStyle = '#fbbf24';
        ctx.setLineDash([4, 4]);
        ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(sumScreen.x, sumScreen.y); ctx.lineTo(trueScreen.x, trueScreen.y); ctx.stroke();
        ctx.setLineDash([]);

        ctx.beginPath(); ctx.arc(trueScreen.x, trueScreen.y, 6, 0, Math.PI * 2);
        ctx.strokeStyle = '#fbbf24'; ctx.lineWidth = 2; ctx.stroke();
        ctx.fillStyle = 'rgba(251, 191, 36, 0.2)'; ctx.fill();
        
        ctx.fillStyle = '#fbbf24';
        ctx.font = '12px sans-serif';
        ctx.fillText('Analytic Value', trueScreen.x + 10, trueScreen.y + 15);
    }

    ctx.fillStyle = '#fff';
    ctx.font = '14px sans-serif';
    const displayVal = showCorrection && mode === 'zeta' ? trueValue : { x: curX, y: curY };
    ctx.fillText(`${mode === 'eta' ? 'η' : 'ζ'}(s) ≈ ${displayVal.x.toFixed(3)} + ${displayVal.y.toFixed(3)}i`, 
        (showCorrection && mode === 'zeta' ? worldToScreen(trueValue.x, trueValue.y, width, height, viewRight).x : sumScreen.x) + 10, 
        (showCorrection && mode === 'zeta' ? worldToScreen(trueValue.x, trueValue.y, width, height, viewRight).y : sumScreen.y));

  }, [params, nTerms, viewRight, mode, showCorrection]);


  // --- Unified Interaction Handlers ---

  const getDistance = (touches) => {
    const dx = touches[0].clientX - touches[1].clientX;
    const dy = touches[0].clientY - touches[1].clientY;
    return Math.sqrt(dx * dx + dy * dy);
  };

  const getCenter = (touches) => {
    return {
      x: (touches[0].clientX + touches[1].clientX) / 2,
      y: (touches[0].clientY + touches[1].clientY) / 2
    };
  };

  const handlePointerDown = (e, panel) => {
    e.preventDefault(); // Prevent default browser drag/select
    const isTouch = e.type === 'touchstart';
    const clientX = isTouch ? e.touches[0].clientX : e.clientX;
    const clientY = isTouch ? e.touches[0].clientY : e.clientY;
    const touches = isTouch ? e.touches : [{ clientX, clientY }];
    
    const canvas = panel === 'left' ? canvasLeftRef.current : canvasRightRef.current;
    const rect = canvas.getBoundingClientRect();
    const x = clientX - rect.left;
    const y = clientY - rect.top;
    
    const view = panel === 'left' ? viewLeft : viewRight;
    const currentInteraction = interactionRef.current[panel];
    
    // Check for Multi-touch (Pinch/Pan)
    if (isTouch && e.touches.length === 2) {
      currentInteraction.mode = 'pinch';
      currentInteraction.startArgs = {
        dist: getDistance(e.touches),
        center: getCenter(e.touches),
        rect: rect
      };
      currentInteraction.startView = { ...view };
      return;
    }

    // Check for Point Drag (Only Left Panel, 1 finger/mouse)
    if (panel === 'left') {
      const worldPos = screenToWorld(x, y, rect.width, rect.height, view);
      const distToPoint = Math.sqrt(
        Math.pow(worldPos.x - params.a, 2) + 
        Math.pow(worldPos.y - params.b, 2)
      );
      
      // Hit test radius in world units approx 20px
      const hitRadius = 20 / view.scale; 
      
      if (distToPoint < hitRadius) {
        currentInteraction.mode = 'drag-point';
        return;
      }
    }

    // Default: Pan
    currentInteraction.mode = 'pan';
    currentInteraction.startArgs = { x: clientX, y: clientY };
    currentInteraction.startView = { ...view };
  };

  const handlePointerMove = (e, panel) => {
    e.preventDefault();
    const isTouch = e.type === 'touchmove';
    const currentInteraction = interactionRef.current[panel];
    const view = panel === 'left' ? viewLeft : viewRight;
    const setView = panel === 'left' ? setViewLeft : setViewRight;
    const canvas = panel === 'left' ? canvasLeftRef.current : canvasRightRef.current;
    const rect = canvas.getBoundingClientRect();

    if (currentInteraction.mode === 'none') return;

    if (currentInteraction.mode === 'drag-point') {
        // Only for Left Panel
        const clientX = isTouch ? e.touches[0].clientX : e.clientX;
        const clientY = isTouch ? e.touches[0].clientY : e.clientY;
        const x = clientX - rect.left;
        const y = clientY - rect.top;
        const worldPos = screenToWorld(x, y, rect.width, rect.height, view);
        setParams({ a: worldPos.x, b: worldPos.y });
    }
    else if (currentInteraction.mode === 'pan') {
        // 1 finger/mouse pan
        const clientX = isTouch ? e.touches[0].clientX : e.clientX;
        const clientY = isTouch ? e.touches[0].clientY : e.clientY;
        
        const start = currentInteraction.startArgs;
        const startView = currentInteraction.startView;
        
        const dx = (clientX - start.x) / startView.scale;
        const dy = (clientY - start.y) / startView.scale; // Screen Y down, World Y up, but pan logic inverts naturally

        setView({
            ...startView,
            x: startView.x - dx,
            y: startView.y + dy 
        });
    }
    else if (currentInteraction.mode === 'pinch' && isTouch && e.touches.length === 2) {
        // 2 finger zoom & pan
        const newDist = getDistance(e.touches);
        const newCenter = getCenter(e.touches);
        const startArgs = currentInteraction.startArgs;
        const startView = currentInteraction.startView;

        // Calculate Scale
        const scaleFactor = newDist / startArgs.dist;
        const newScale = startView.scale * scaleFactor;

        // Calculate Pan (Center shift)
        // We want the point under the startCenter (in world coords) to remain at the newCenter
        // 1. Get world coord of startCenter using startView
        const startCenterRelX = startArgs.center.x - startArgs.rect.left;
        const startCenterRelY = startArgs.center.y - startArgs.rect.top;
        
        const worldFocus = screenToWorld(
            startCenterRelX, 
            startCenterRelY, 
            startArgs.rect.width, 
            startArgs.rect.height, 
            startView
        );

        // 2. Adjust View Center
        // New Screen Pos = (World - ViewX) * Scale + Offset
        // We want Screen Pos to be newCenterRel
        // ViewX = World - (Screen - Offset) / Scale
        
        const newCenterRelX = newCenter.x - startArgs.rect.left;
        const newCenterRelY = newCenter.y - startArgs.rect.top;

        const newViewX = worldFocus.x - (newCenterRelX - startArgs.rect.width/2) / newScale;
        const newViewY = worldFocus.y + (newCenterRelY - startArgs.rect.height/2) / newScale;

        setView({
            scale: newScale,
            x: newViewX,
            y: newViewY
        });
    }
  };

  const handlePointerUp = (panel) => {
    interactionRef.current[panel].mode = 'none';
  };

  const handleWheel = (e, panel) => {
      e.preventDefault();
      const setView = panel === 'left' ? setViewLeft : setViewRight;
      
      setView(prev => {
        const zoomIntensity = 0.001;
        const scaleFactor = Math.exp(-e.deltaY * zoomIntensity);
        return {
            ...prev,
            scale: prev.scale * scaleFactor
        };
      });
  };

  // --- Effects ---
  useEffect(() => {
    const handleResize = () => {
      if (leftContainerRef.current && canvasLeftRef.current) {
        canvasLeftRef.current.width = leftContainerRef.current.clientWidth;
        canvasLeftRef.current.height = leftContainerRef.current.clientHeight;
        drawLeft();
      }
      if (rightContainerRef.current && canvasRightRef.current) {
        canvasRightRef.current.width = rightContainerRef.current.clientWidth;
        canvasRightRef.current.height = rightContainerRef.current.clientHeight;
        drawRight();
      }
    };
    window.addEventListener('resize', handleResize);
    handleResize(); 
    return () => window.removeEventListener('resize', handleResize);
  }, [drawLeft, drawRight]);

  useEffect(() => { drawLeft(); drawRight(); }, [drawLeft, drawRight]);

  return (
    <div className="flex flex-col h-screen bg-slate-950 text-slate-200 font-sans overflow-hidden">
      
      {/* Header */}
      <header className="flex-none p-4 bg-slate-900 border-b border-slate-800 flex flex-wrap items-center justify-between gap-4">
        <div className="flex items-center gap-3">
            <div className="w-8 h-8 rounded bg-gradient-to-br from-cyan-500 to-blue-600 flex items-center justify-center font-bold text-white italic">
                ζ
            </div>
            <h1 className="text-xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-blue-400 to-cyan-300">
                Riemann Zeta Visualizer
            </h1>
        </div>

        {/* Controls */}
        <div className="flex items-center gap-6">
            <div className="flex flex-col w-48">
                <div className="flex justify-between text-xs text-slate-400 mb-1">
                    <span>Terms (N)</span>
                    <span>{nTerms}</span>
                </div>
                <input type="range" min="1" max="1000" value={nTerms} onChange={(e) => setNTerms(parseInt(e.target.value))}
                    className="h-2 bg-slate-700 rounded-lg appearance-none cursor-pointer accent-blue-500" />
            </div>

            <div className="flex items-center gap-2">
                 <button 
                    onClick={() => setShowCorrection(!showCorrection)}
                    className={`flex items-center gap-2 px-3 py-1.5 rounded-md text-xs border ${showCorrection ? 'bg-amber-900/30 border-amber-600 text-amber-200' : 'bg-slate-800 border-slate-700 text-slate-400'}`}
                    title="Visualize Euler-Maclaurin Correction Term"
                >
                    <Calculator size={14} />
                    Analytic Corr.
                </button>
            </div>

            <div className="flex items-center bg-slate-800 p-1 rounded-lg border border-slate-700">
                <button onClick={() => setMode('zeta')} className={`px-3 py-1.5 rounded-md text-sm transition-colors ${mode === 'zeta' ? 'bg-cyan-600 text-white shadow' : 'text-slate-400 hover:text-white'}`}>
                    Standard ζ(s)
                </button>
                <button onClick={() => setMode('eta')} className={`px-3 py-1.5 rounded-md text-sm transition-colors ${mode === 'eta' ? 'bg-emerald-600 text-white shadow' : 'text-slate-400 hover:text-white'}`}>
                    Dirichlet η(s)
                </button>
            </div>
        </div>
      </header>

      {/* Main Content */}
      <div className="flex-1 flex flex-col md:flex-row overflow-hidden">
        
        {/* Left Panel */}
        <div className="relative flex-1 border-r border-slate-800 flex flex-col min-h-[300px] touch-none">
            <div className="absolute top-4 left-4 z-10 bg-slate-900/80 backdrop-blur p-2 rounded border border-slate-700 text-xs pointer-events-none select-none">
                <div className="font-bold text-slate-300 mb-1">Input Space (s-plane)</div>
                <div className="text-blue-400">Re(s) = {params.a.toFixed(3)}</div>
                <div className="text-purple-400">Im(s) = {params.b.toFixed(3)}</div>
                <div className="mt-1 text-slate-500 flex items-center gap-1">
                   <MousePointer2 size={10} /> Drag point to move s
                </div>
                <div className="text-slate-500 flex items-center gap-1">
                   <Move size={10} /> Drag background to pan
                </div>
            </div>
            <div 
                ref={leftContainerRef} 
                className="w-full h-full cursor-crosshair touch-none"
                onMouseDown={(e) => handlePointerDown(e, 'left')}
                onMouseMove={(e) => handlePointerMove(e, 'left')}
                onMouseUp={() => handlePointerUp('left')}
                onMouseLeave={() => handlePointerUp('left')}
                onTouchStart={(e) => handlePointerDown(e, 'left')}
                onTouchMove={(e) => handlePointerMove(e, 'left')}
                onTouchEnd={() => handlePointerUp('left')}
                onWheel={(e) => handleWheel(e, 'left')}
            >
                <canvas ref={canvasLeftRef} className="block w-full h-full touch-none" />
            </div>
        </div>

        {/* Right Panel */}
        <div className="relative flex-1 flex flex-col min-h-[300px] bg-slate-950 touch-none">
            <div className="absolute top-4 left-4 z-10 bg-slate-900/80 backdrop-blur p-2 rounded border border-slate-700 text-xs shadow-lg pointer-events-none select-none">
                <div className="font-bold text-slate-300 mb-1">Output Space</div>
                <div className="text-slate-400">Showing {nTerms} vectors</div>
                 {showCorrection && mode === 'zeta' && (
                    <div className="text-amber-400 mt-1 font-semibold">
                        Corrected via Euler-Maclaurin
                    </div>
                )}
            </div>

            <div className="absolute top-4 right-4 z-10 flex flex-col gap-2">
                <button onClick={() => setViewRight(prev => ({ ...prev, scale: prev.scale * 1.2 }))} className="p-2 bg-slate-800 hover:bg-slate-700 rounded border border-slate-600 text-white"><ZoomIn size={16} /></button>
                <button onClick={() => setViewRight(prev => ({ ...prev, scale: prev.scale * 0.8 }))} className="p-2 bg-slate-800 hover:bg-slate-700 rounded border border-slate-600 text-white"><ZoomOut size={16} /></button>
                <button onClick={() => setViewRight({ x: 0, y: 0, scale: 100 })} className="p-2 bg-slate-800 hover:bg-slate-700 rounded border border-slate-600 text-white"><RefreshCw size={16} /></button>
            </div>
            
            <div 
                ref={rightContainerRef} 
                className="w-full h-full cursor-move touch-none"
                onMouseDown={(e) => handlePointerDown(e, 'right')}
                onMouseMove={(e) => handlePointerMove(e, 'right')}
                onMouseUp={() => handlePointerUp('right')}
                onMouseLeave={() => handlePointerUp('right')}
                onTouchStart={(e) => handlePointerDown(e, 'right')}
                onTouchMove={(e) => handlePointerMove(e, 'right')}
                onTouchEnd={() => handlePointerUp('right')}
                onWheel={(e) => handleWheel(e, 'right')}
            >
                <canvas ref={canvasRightRef} className="block w-full h-full touch-none" />
            </div>
        </div>
      </div>

      <footer className="bg-slate-900 border-t border-slate-800 p-3 text-xs text-slate-500 flex justify-center items-center gap-6 select-none">
          <div className="flex items-center gap-2"><span className="w-3 h-3 bg-blue-500 rounded-full"></span><span>Input (s)</span></div>
          <div className="flex items-center gap-2"><span className={`w-3 h-3 ${mode === 'zeta' ? 'bg-cyan-500' : 'bg-emerald-500'} rounded-full`}></span><span>Partial Sum</span></div>
          {showCorrection && mode === 'zeta' && (
              <div className="flex items-center gap-2"><span className="w-3 h-3 border-2 border-amber-500 rounded-full"></span><span className="text-amber-500">Analytic Value</span></div>
          )}
      </footer>
    </div>
  );
};

export default App;
