`include "config.svh"

module lab_top
# (
    parameter  clk_mhz       = 50,
               w_key         = 4,
               w_sw          = 8,
               w_led         = 8,
               w_digit       = 8,
               w_gpio        = 100,

               screen_width  = 640,
               screen_height = 480,

               w_red         = 4,
               w_green       = 4,
               w_blue        = 4,

               w_x           = $clog2 ( screen_width  ),
               w_y           = $clog2 ( screen_height )
)
(
    input                        clk,
    input                        slow_clk,
    input                        rst,

    // Keys, switches, LEDs

    input        [w_key   - 1:0] key,
    input        [w_sw    - 1:0] sw,
    output logic [w_led   - 1:0] led,

    // A dynamic seven-segment display

    output logic [          7:0] abcdefgh,
    output logic [w_digit - 1:0] digit,

    // Graphics

    input        [w_x     - 1:0] x,
    input        [w_y     - 1:0] y,

    output logic [w_red   - 1:0] red,
    output logic [w_green - 1:0] green,
    output logic [w_blue  - 1:0] blue,

    // Microphone, sound output and UART

    input        [         23:0] mic,
    output       [         15:0] sound,

    input                        uart_rx,
    output                       uart_tx,

    // General-purpose Input/Output

    inout        [w_gpio  - 1:0] gpio
);

   
    wire [w_digit - 1:0] dots = '0;
    localparam w_number = w_digit * 4;

    // seven_segment_display # (w_digit)
    // i_7segment (.number (w_number' (mic)), .*);

    //------------------------------------------------------------------------
    //
    //  Measuring frequency
    //
    //------------------------------------------------------------------------

    // It is enough for the counter to be 20 bit. Why?

    logic [23:0] prev_mic;
    logic [19:0] counter;
    logic [19:0] distance;

    logic pulse;

    strobe_gen # (.clk_mhz (27), .strobe_hz (30))
    i_strobe_gen (clk, rst, pulse);

    // Player positions
    logic [8:0] move_bx = 100, move_by = 100;
    logic [8:0] move_gx = 200, move_gy = 100;

    // Scores
    logic [3:0] b_score = 0, g_score = 0;

      // Red squares (targets)
    logic [8:0] red_x[6:0] = '{34, 60, 172, 180, 255, 350, 400};
    logic [8:0] red_y[6:0] = '{200, 60, 204, 50, 130, 230, 28};
    logic       red_fixed[6:0]  =  '{0, 0, 0, 0, 0, 0, 0};


    // Sizes
    localparam blue_w = 30, blue_h = 30;
    localparam green_w = 30, green_h = 30;
    localparam red_w = 10, red_h = 10;

// Display logic
    always_comb begin
        red = 0;
        green = 0;
        blue = 0;

        // Draw red square
        for (int i = 0; i < 7; i++) 
        begin
            if (red_fixed[i]) 
            begin
                if (x >= red_x[i] && x < red_x[i] + red_w &&
                    y >= red_y[i] && y < red_y[i] + red_h)
                    red = 31;
            end
        end

        // Draw blue square
        if (x >= move_bx && x < move_bx + blue_w &&
            y >= move_by && y < move_by + blue_h)
            blue = 50;

        // Draw green square
        if (x >= move_gx && x < move_gx + green_w &&
            y >= move_gy && y < move_gy + green_h)
            green = 63;

        // Winner
        if ((b_score + g_score == 4) && (b_score == g_score)) begin
            red = 31;
            green = 0;
            blue = 0;
        end
        else if ((b_score + g_score == 4) && (b_score > g_score)) begin
            red = 0;
            green = 0;
            blue = 31;
        end
        else if ((b_score + g_score == 4) && (g_score > b_score)) begin
            red = 0;
            green = 63;
            blue = 0;
        end
    end

// Movement and collision logic
    always_ff @(posedge pulse or posedge rst) 
    begin
        if (rst) 
        begin
            move_bx <= 100; move_by <= 100;
            move_gx <= 200; move_gy <= 100;
            b_score <= 0; g_score <= 0;
            for (int i = 0; i < 7; i++) 
            begin
                red_fixed[i] <= 0;
                
            end
        end 
    
        else
         begin
            // Activate red square when a specific note is detected
            if (t_note == A)  red_fixed[0] <= 1;
            if (t_note == B)  red_fixed[1] <= 1;
            if (t_note == C)  red_fixed[2] <= 1; 
            if (t_note == D)  red_fixed[3] <= 1; 
            if (t_note == E)  red_fixed[4] <= 1; 
            if (t_note == F)  red_fixed[5] <= 1; 
            if (t_note == G)  red_fixed[6] <= 1; 

            // Move Blue square
            if (key[0] && move_bx < screen_width - blue_w)  move_bx <= move_bx + 1;
            if (key[1] && move_bx > 0)                      move_bx <= move_bx - 1;
            if (key[2] && move_by < screen_height - blue_h) move_by <= move_by + 1;
            if (key[3] && move_by > 0)                      move_by <= move_by - 1;

            // Move Green square
            if (key[4] && move_gx < screen_width - green_w) move_gx <= move_gx + 1;
            if (key[5] && move_gx > 0)                      move_gx <= move_gx - 1;
            if (key[6] && move_gy < screen_height - green_h) move_gy <= move_gy + 1;
            if (key[7] && move_gy > 0)                      move_gy <= move_gy - 1;

            // Collision detection
            for (int i = 0; i < 7; i++) 
            begin
                if (red_fixed[i]) 
                begin
                    // Green touches red 
                    if (
                        move_gx + green_w > red_x[i] &&
                        move_gx < red_x[i] + red_w &&
                        move_gy + green_h > red_y[i] &&
                        move_gy < red_y[i] + red_h
                    ) 
                    begin
                        g_score <= g_score + 1;
                        red_fixed[i] <= 0;
                    end
                end
                    // Blue touches red 
                if (red_fixed[i] ) 
                begin
                    if (
                        move_bx + blue_w > red_x[i] &&
                        move_bx < red_x[i] + red_w &&
                        move_by + blue_h > red_y[i] &&
                        move_by < red_y[i] + red_h
                    ) 
                    begin
                        b_score <= b_score + 1;
                        red_fixed[i] <= 0;
                    end
                end
            end

        end
    end
    

    always_ff @ (posedge clk or posedge rst)
        if (rst)
        begin
            prev_mic <= '0;
            counter  <= '0;
            distance <= '0;
        end
        else
        begin
            prev_mic <= mic;

            // Crossing from negative to positive numbers

            if (  prev_mic [$left ( prev_mic )] == 1'b1
                & mic      [$left ( mic      )] == 1'b0 )
            begin
               distance <= counter;
               counter  <= 20'h0;
            end
            else if (counter != ~ 20'h0)  // To prevent overflow
            begin
               counter <= counter + 20'h1;
            end
        end

    
    `ifdef USE_STANDARD_FREQUENCIES

    localparam freq_100_C  = 26163,
               freq_100_Cs = 27718,
               freq_100_D  = 29366,
               freq_100_Ds = 31113,
               freq_100_E  = 32963,
               freq_100_F  = 34923,
               freq_100_Fs = 36999,
               freq_100_G  = 39200,
               freq_100_Gs = 41530,
               freq_100_A  = 44000,
               freq_100_As = 46616,
               freq_100_B  = 49388;
    `else

    // Custom measured frequencies

    localparam freq_100_C  = 26163,
               freq_100_Cs = 27718,
               freq_100_D  = 29366,
               freq_100_Ds = 31113,
               freq_100_E  = 32963,
               freq_100_F  = 34923,
               freq_100_Fs = 36999,
               freq_100_G  = 39200,
               freq_100_Gs = 41530,
               freq_100_A  = 44000,
               freq_100_As = 46616,
               freq_100_B  = 49388;
    `endif

    //------------------------------------------------------------------------

    function [19:0] high_distance (input [18:0] freq_100);
       high_distance = clk_mhz * 1000 * 1000 / freq_100 * 103;
    endfunction

    //------------------------------------------------------------------------

    function [19:0] low_distance (input [18:0] freq_100);
       low_distance = clk_mhz * 1000 * 1000 / freq_100 * 97;
    endfunction

    //------------------------------------------------------------------------

    function [19:0] check_freq_single_range (input [18:0] freq_100, input [19:0] distance);

       check_freq_single_range =    distance > low_distance  (freq_100)
                                  & distance < high_distance (freq_100);
    endfunction

    //------------------------------------------------------------------------

    function [19:0] check_freq (input [18:0] freq_100, input [19:0] distance);

       check_freq =   check_freq_single_range (freq_100 * 4 , distance)
                    | check_freq_single_range (freq_100 * 2 , distance)
                    | check_freq_single_range (freq_100     , distance);

    endfunction

    //------------------------------------------------------------------------

    wire check_C  = check_freq (freq_100_C  , distance );
    wire check_Cs = check_freq (freq_100_Cs , distance );
    wire check_D  = check_freq (freq_100_D  , distance );
    wire check_Ds = check_freq (freq_100_Ds , distance );
    wire check_E  = check_freq (freq_100_E  , distance );
    wire check_F  = check_freq (freq_100_F  , distance );
    wire check_Fs = check_freq (freq_100_Fs , distance );
    wire check_G  = check_freq (freq_100_G  , distance );
    wire check_Gs = check_freq (freq_100_Gs , distance );
    wire check_A  = check_freq (freq_100_A  , distance );
    wire check_As = check_freq (freq_100_As , distance );
    wire check_B  = check_freq (freq_100_B  , distance );

    //------------------------------------------------------------------------

    localparam w_note = 12;

    wire [w_note - 1:0] note = { check_C  , check_Cs , check_D  , check_Ds ,
                                 check_E  , check_F  , check_Fs , check_G  ,
                                 check_Gs , check_A  , check_As , check_B  };

    localparam [w_note - 1:0] no_note = 12'b0,

                              C  = 12'b1000_0000_0000,
                              Cs = 12'b0100_0000_0000,
                              D  = 12'b0010_0000_0000,
                              Ds = 12'b0001_0000_0000,
                              E  = 12'b0000_1000_0000,
                              F  = 12'b0000_0100_0000,
                              Fs = 12'b0000_0010_0000,
                              G  = 12'b0000_0001_0000,
                              Gs = 12'b0000_0000_1000,
                              A  = 12'b0000_0000_0100,
                              As = 12'b0000_0000_0010,
                              B  = 12'b0000_0000_0001;

    localparam [w_note - 1:0] Df = Cs, Ef = Ds, Gf = Fs, Af = Gs, Bf = As;

    //------------------------------------------------------------------------
    //
    //  Note filtering
    //
    //------------------------------------------------------------------------

    logic  [w_note - 1:0] d_note;  // Delayed note

    always_ff @ (posedge clk or posedge rst)
        if (rst)
            d_note <= no_note;
        else
            d_note <= note;

    logic  [19:0] t_cnt;           // Threshold counter
    logic  [w_note - 1:0] t_note;  // Thresholded note

    always_ff @ (posedge clk or posedge rst)
        if (rst)
            t_cnt <= 0;
        else
            if (note == d_note)
                t_cnt <= t_cnt + 1;
            else
                t_cnt <= 0;

    always_ff @ (posedge clk or posedge rst)
    begin
        if (rst)
            t_note <= no_note;
        else
            if (& t_cnt)
                t_note <= d_note; 
    end
    //------------------------------------------------------------------------
    //
    //  The output to seven segment display
    //
    //------------------------------------------------------------------------

    always_ff @ (posedge clk or posedge rst)
        if (rst)
            abcdefgh <= 8'b00000000;
        else
            case (t_note)
            C  : abcdefgh <= 8'b10011100;  // C   // abcdefgh
            Cs : abcdefgh <= 8'b10011101;  // C#
            D  : abcdefgh <= 8'b01111010;  // D   //   --a--
            Ds : abcdefgh <= 8'b01111011;  // D#  //  |     |
            E  : abcdefgh <= 8'b10011110;  // E   //  f     b
            F  : abcdefgh <= 8'b10001110;  // F   //  |     |
            Fs : abcdefgh <= 8'b10001111;  // F#  //   --g--
            G  : abcdefgh <= 8'b10111100;  // G   //  |     |
            Gs : abcdefgh <= 8'b10111101;  // G#  //  e     c
            A  : abcdefgh <= 8'b11101110;  // A   //  |     |
            As : abcdefgh <= 8'b11101111;  // A#  //   --d--  h
            B  : abcdefgh <= 8'b00111110;  // B
            default : abcdefgh <= 8'b00000010;
            endcase

    assign digit = w_digit' (1);

    //------------------------------------------------------------------------
    //
    //  Exercise 4: Replace filtered note with unfiltered note.
    //  Do you see the difference?
    //
    //------------------------------------------------------------------------

endmodule
