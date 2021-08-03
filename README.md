# frequency-meter-fpga-four-7-segments-display
frequency meter, frequency counter FPGA with four 7 segments displays in behavioral language verilog HDL



O código tem always do tipo sequencial, tem always do tipo combinacional, e uma task.
* Lembrando que cada segmento do display é ativo em nível baixo.



## Código-fonte:

```verilog


module frequecymeter ( input clock, clear, signal_in, 
					   output reg a1, b1, c1, d1, e1, f1, g1,
					   output reg a2, b2, c2, d2, e2, f2, g2,
					   output reg a3, b3, c3, d3, e3, f3, g3,
					   output reg a4, b4, c4, d4, e4, f4, g4,
					   output reg[9:0] freq_out
					   );  



reg signal_in_last, signal_in_last1, edge_detected, signal_in_prev;   //do prof


// serão flip flops
reg bandeira,             //para nova contagem após 1 segundo

	bandeira2,       //para o display das dezenas
	bandeira3,          //para o display das centenas
	bandeira4,        //para o display dos milhares
	bandeira5,          //para atualizar as saídas  (tanto dos displays quanto do freq_out)
	bandeira6,         //para tolerância do 1MHz
	bandeira7,                //para tolerância do 100kHz
	bandeira8,          //para o Hi
	bandeira9,           //para o Lo
	bandeira_contagem;    //para a contagem do valor que vai aparecer no display   , além do display das unidades
	
reg [25:0]clock_count;   // contador de clock
reg [9:0]frequencia;  // apenas a porção milhar da frequencia legítima
reg [25:0]freq_grande;  //frequencia legítima
	
	
reg [3:0] display1, display2, display3, display4;    //os valores que serão apresentados nos displays




// serão lookup tables
reg flag,                 //para nova contagem após 1 segundo
	
	flag2,                //para o display das dezenas
	flag3,               //para o display das centenas
	flag4,                 //para o display dos milhares
	flag5,               //para atualizar as saídas  (tanto dos displays quanto do freq_out)
	flag6,                 //para tolerância do 1MHz
	flag7,              //para tolerância do 100kHz
	flag8,                 //para o Hi
	flag9,                 //para o Lo
	flag_contagem;          //para a contagem do valor que vai aparecer no display  , além do display das unidades
	


initial begin
	
	

	freq_grande <= 26'b0;
	 
	freq_out <= 10'd0;
	
	display1 <= 4'd0; 
	display2 <= 4'd0; 
	display3 <= 4'd0; 
	display4 <= 4'd0;
	
	
	bandeira <= 1'b0;             //para nova contagem após 1 segundo

	bandeira2 <= 1'b0;       //para o display das dezenas
	bandeira3 <= 1'b0;          //para o display das centenas
	bandeira4 <= 1'b0;        //para o display dos milhares
	bandeira5 <= 1'b0;          //para atualizar as saídas  (tanto dos displays quanto do freq_out)
	bandeira6 <= 1'b0;         //para tolerância do 1MHz
	bandeira7 <= 1'b0;                //para tolerância do 100kHz
	bandeira8 <= 1'b0;          //para o Hi
	bandeira9 <= 1'b0;           //para o Lo
	bandeira_contagem <= 1'b0;    //para a contagem do valor que vai aparecer no display   , além do display das unidades
	
	clock_count <= 26'd0;           //conta o número de clocks para ver quando será 50M clocks e, a partir daí, zeras os parâmetros apropriados e renovar a contagem.
	frequencia <= 10'd0; 
	freq_grande <=26'd0;  //frequencia legítima
	
	
	
	
	
	flag = 1'b0;                 //para nova contagem após 1 segundo

	flag2 = 1'b0;                //para o display das dezenas
	flag3 = 1'b0;               //para o display das centenas
	flag4 = 1'b0;                 //para o display dos milhares
	flag5 = 1'b0;               //para atualizar as saídas  (tanto dos displays quanto do freq_out)
	flag6 = 1'b0;                 //para tolerância do 1MHz
	flag7 = 1'b0;              //para tolerância do 100kHz
	flag8 = 1'b0;                 //para o Hi
	flag9 = 1'b0;                 //para o Lo
	flag_contagem = 1'b0;          //para a contagem do valor que vai aparecer no display
	
	
	{a1, b1, c1, d1, e1, f1, g1} = 7'b0000_001;   //unidade
	{a2, b2, c2, d2, e2, f2, g2} = 7'b0000_001;   //dezena
	{a3, b3, c3, d3, e3, f3, g3} = 7'b0000_001;   //centena
	{a4, b4, c4, d4, e4, f4, g4} = 7'b0000_001;   //milhar
	
	
end


task apresenta (input [3:0]valor, output a5, b5, c5, d5, e5, f5, g5);
	case (valor)
		4'd0: {a5, b5, c5, d5, e5, f5, g5} = 7'b0000_001;   //0
		4'd1: {a5, b5, c5, d5, e5, f5, g5} = 7'b1001_111;   //1
		4'd2: {a5, b5, c5, d5, e5, f5, g5} = 7'b0010_010;   //2 
		4'd3: {a5, b5, c5, d5, e5, f5, g5} = 7'b0000_110;   //3
		4'd4: {a5, b5, c5, d5, e5, f5, g5} = 7'b1001_100;   //4 
		4'd5: {a5, b5, c5, d5, e5, f5, g5} = 7'b0100_100;   //5
		4'd6: {a5, b5, c5, d5, e5, f5, g5} = 7'b0100_000;   //6
		4'd7: {a5, b5, c5, d5, e5, f5, g5} = 7'b0001_111;   //7
		4'd8: {a5, b5, c5, d5, e5, f5, g5} = 7'b0000_000;   //8
		4'd9: {a5, b5, c5, d5, e5, f5, g5} = 7'b0000_100;   //9
		
		4'd10: {a5, b5, c5, d5, e5, f5, g5} = 7'b1111_110;   //-
		4'd11: {a5, b5, c5, d5, e5, f5, g5} = 7'b1110_001;   //L
		4'd12: {a5, b5, c5, d5, e5, f5, g5} = 7'b1001_000;   //H
		4'd13: {a5, b5, c5, d5, e5, f5, g5} = 7'b1100_010;   //o (a letra "o" de óculos)
		4'd14: {a5, b5, c5, d5, e5, f5, g5} = 7'b0000_000;   //i
		
		default: {a5, b5, c5, d5, e5, f5, g5} = 7'b1111_111;  
	endcase
endtask



always @(posedge clock or negedge clear) begin   //sincronizador do prof

	if(!clear)begin
		signal_in_last <= 1'b0;
		signal_in_last1 <= 1'b0;
	end
	else begin
		signal_in_last <= signal_in;
		signal_in_last1 <= signal_in_last;
	end

end

always @(posedge clock or negedge clear) begin    //detector do prof

if(!clear) begin
		edge_detected <= 1'b0;
		signal_in_prev <= 1'b0;
	end
	else if( signal_in_prev == 1'b0 && signal_in_last1 == 1'b1) begin
		edge_detected <= 1'b1;
		signal_in_prev <= signal_in_last1;
	end
	else begin
		edge_detected <= 1'b0;
		signal_in_prev <= signal_in_last1;
	end
	
end












//clock_count    contador                #circuitosequencial    #contagemdoclock_count
always @(posedge clock or negedge clear or posedge flag) begin
	if ((clear == 1'b0)|(flag == 1'b0))   //se resetado ou nova contagem
		clock_count <= 26'd0;
	else
		clock_count <= clock_count + 26'd1;    //clock_count se torna flip flop.
	
	
	//apenas para por um flip flop
	bandeira <= flag;  //inicia uma nova contagem
 
	bandeira2 <= flag2;    // display dezena
	bandeira3 <= flag3;     //display centena
	bandeira4 <= flag4;   //display milhar
	bandeira5 <= flag5;   //apresenta o valor nas saídas
	bandeira6 <= flag6;    //para tolerância do 1MHz
	bandeira7 <= flag7;    //para tolerância do 100kHz
	bandeira8 <= flag8;       //para o Hi
	bandeira9 <= flag9;     //para o Lo  
	bandeira_contagem <= flag_contagem;     //para o incremento da "frequencia"   
	
	
end


//clock_count   julgamentos         #circuitocombinacional    #julgamentodoclock_count
always @(clock_count)begin
	
	if(clock_count == 26'd50_000_000)begin    //já passou 1 segundo?    //Origial..........   // O Grande Julgamento...........................................................
		if(flag6 || flag7 || flag8 || flag9)begin
			if(flag6)
				flag6 = 1'b0;    //gerando negsedge do 1MHz   //o que acarreta na apresentação do 
			else if(flag7)
				flag7 = 1'b0;    //gerando negsedge do 100KHz  //o que acarreta na apresentação do
			else if(flag8)
				flag8 = 1'b0;    //gerando negsedge do Hi    //o que acarreta na apresentação do
			else
				flag9 = 1'b0;	 //gerando negsedge do Lo   //o que acarreta na apresentação do
		end
		else    
			flag5 = 1'b1;   //gerando porsedge  //apresenta o valor da contagem da freq_saida nos displays  //Apresenta o Original...
		//os parâmetros primeiros são apresentados, aí no ciclo seguinte é que são zerados (sem zerar as saídas) (sem mexer na última posição deixada dos disjuntores).
	end
	else if(clock_count == 26'd50_000_001) begin
		flag = 1'b1;  //zera os parâmetros pra uma nova contagem   dafdasdfafdafddasfgfdgfasfa  e como fica o clock_count =0  ou =1, acho melhor 1, ai vai ter que ajeita o 50M
		flag5 = 1'b0;
	end
	else 
		flag = 1'b0;  
end










//freq_grande   contador    #sequencial    #frequencialegítimagrandona
always @(posedge edge_detected or negedge clear or posedge bandeira)begin
	if(!clear)
		freq_grande <= 26'd0; 
	else if(flag)   //iniciar nova contagem
		freq_grande <= 26'd0;  	
	else //então foi o edge_detected
		freq_grande <= freq_grande + 26'd1;
end



//freq_grande   contagem   flag6, flag7, flag8, flag9   julgamentos     #combinacional    #frequenciacompleta        
always @(freq_grande)begin
	if( (freq_grande>=26'd100_000) && (freq_grande<=1_000_000))begin 
	
		if(freq_grande)begin
			if(!(freq_grande%26'd1_000))begin
				flag_contagem = 1'b1;      //gerando um posedge nessa bandeira contagem.
			end
			else begin
				flag_contagem = 1'b0;
			end
		end
		else begin
			flag_contagem = 1'b0;   //caso ocorra algum clear aí, no meio do caminho.
		end
		
		flag7 = 1'b0;  //já que a a contagem da fequencia teve de passar por valores menores, e na hora do Grande Julgamento, nada será percebido quanto a isso.
		flag9 = 1'b0;  //já que a a contagem da fequencia teve de passar por valores menores, e na hora do Grande Julgamento, nada será percebido quanto a isso.
	end
	else if(freq_grande > 26'd1_000_500)begin
		flag8 = 1'b1;   //para o Hi
	end
	else if (freq_grande < 10'd99_500)begin
		flag9 = 1'b1;  //para o Lo
	end
	else if( freq_grande <  26'd100_000 )begin
		flag7 = 1'b1;   //para fixar em 100kHz 
	end
	else if (freq_grande > 10'd1_000_000)begin
		flag6 = 1'b1;  //para fixar em 1000kHz
	end


end






//frequencia    saída freq_out     contador           #sequencial    #frequenciaseleta   
							//apresetaação do freq_out   flag6, flag7, flag8 e flag9
always @(posedge bandeira_contagem  or negedge clear or posedge bandeira or posedge bandeira5 or negedge bandeira6 or negedge bandeira7 or negedge bandeira8 or negedge bandeira9) begin
	
	if(!clear)begin
		freq_out <= 10'd0;  //apresenta 0KHz na saída freq_out
		frequencia <= 10'd0;
	end
	else if(bandeira)   //iniciar nova contagem  //não se zera o fre_count nesse caso.
		frequencia <= 10'd0;   
	else if(bandeira5)  //apresenta Original  na saída freq_out
		freq_out <= frequencia;  
	else if(bandeira6)   //apresenta 1_000 KHz  na saída freq_out por tolerancia
		freq_out <= 10'd1_000;  
	else if(bandeira7)   //apresenta 100 KHz  na saída freq_out  por tolerancia
		freq_out <= 10'd100;  
	else if(bandeira8)   //apresenta 0KHz na saída freq_out   
		freq_out <= 10'd0;  
	else if(bandeira9)  //apresenta 0KHz na saída freq_out  
		freq_out <= 10'd0;  
	else //então foi o posedge do bandeira_contagem    "o Original em atividade"
		frequencia <= frequencia + 10'd1;       //isso é importante, pois servirá para dar valor ao freq_out, cuja dimensão é de 10bits.


end








//****************************displays*************************************************************************************************


//para o display das unidades     #sequencial 
always @(posedge bandeira_contagem or negedge clear or posedge bandeira5)begin 
	if(!clear)
		apresenta(4'd0, a1, b1, c1, d1, e1, f1, g1);
	else if (bandeira5)
		apresenta(display1, a1, b1, c1, d1, e1, f1, g1);
	else begin //então foi o bandeira_contagem
		if(display1 == 4'd9)begin
			display1 <= 4'd0;
			bandeira2 <= 1'b1;            
		end
		else 
			display1 <= display1 + 4'd1;
	end
end


//para o display das dezenas      #sequencial
always @(posedge bandeira2 or negedge clear or posedge bandeira5)begin 
	if(!clear)
		apresenta(4'd0, a2, b2, c2, d2, e2, f2, g2);
	else if (bandeira5)
		apresenta(display2, a2, b2, c2, d2, e2, f2, g2);
	else begin //então foi o bandeira2
		if(display2 == 4'd9)begin
			display2 <= 4'd0;
			bandeira3 <= 1'b1;
		end
		else 
			display2 <= display2 + 4'd1;
	end
end


//para o display das centenas      #sequencial
always @(posedge bandeira3 or negedge clear or posedge bandeira5)begin 
	if(!clear)begin
		apresenta(4'd0, a3, b3, c3, d3, e3, f3, g3);
	end
	else if (bandeira5)begin
		apresenta(display3, a3, b3, c3, d3, e3, f3, g3);
	end
	else begin //então foi o bandeira3 begin
		if(display3 == 4'd9)begin
			display3 <= 4'd0;
			bandeira4 <= 1'b1;
		end
		else begin
			display3 <= display3 + 4'd1;
		end
	end
end


//para o display das milhares      #sequencial
always @(posedge bandeira4 or negedge clear or posedge bandeira5)begin 
	if(!clear)
		apresenta(4'd0, a4, b4, c4, d4, e4, f4, g4);
	else if (bandeira5)
		apresenta(display1, a4, b4, c4, d4, e4, f4, g4);
	else //então foi o bandeira4
		display4 <= display4 + 4'd1;
end




endmodule

```



Observações sobre o código:
* Como o clock é de 50 MHz, a cada 50_000_000 (cinquenta milhoes) de posedge do clock teremos 1 segundo.
* Uma frequencia de 100kHz são 100_000 (cem mil) posedge de edge_detected dentro de 1 segundo, ou seja, dentro de 50_000_000 (cinquenta milhoes) de posedge do clock.
* Nesse código, eu fiz com que frequências de 99_500 Hz a 100_000 fossem iguais a 100kHz no display (display apresenta 0100).
* Eu também fiz com que frequências de 1_000_000 a 1_000_500 fosse iguais a 1000kHz no display (display apresenta 1000).





















