# software-de-controle-de-velocidade-de-biorreator-de-celulas-animais

#include <msp430g2553.h>

unsigned int velocidade = 8000;
unsigned int vel_maxima = 20000;

void horario(void)
{
   TA1CCR1=velocidade;
   TA1CCR2=0;
}

void anti_horario(void)
{
   TA1CCR1=0;
   TA1CCR2=velocidade;
}

void parar(void)
{
   TA1CCR1=0;
   TA1CCR2=0;
}

void main( void )
{
   // Stop watchdog timer to prevent time out reset
   WDTCTL = WDTPW + WDTHOLD;
   BCSCTL1 = CALBC1_1MHZ;
   DCOCTL = CALDCO_1MHZ;
   P1DIR = 48;
   P1OUT=0;
   P1DIR |= BIT0;
   P1OUT &= ~BIT0;
   P1DIR |= BIT1;//TX
   P1DIR &=~BIT2;//RX
   P1SEL |= BIT1+BIT2;
   P1SEL2 |= BIT1+BIT2;
   UCA0CTL1 |= UCSSEL_2; //SMCLK
   UCA0BR0 = 104;
   UCA0BR1 = 0;
   UCA0MCTL = UCBRS_0;
   UCA0CTL1 &= ~UCSWRST;
   IE2 |= UCA0RXIE;
   // Inicialização do PWM
   P2DIR |= BIT1+BIT5;
   P2SEL |= BIT1+BIT5;
   //P2SEL2 |= BIT1+BIT5;
   TA1CCR0 = 20000;                             // Período do PWM --> ~50Hz
   TA1CCTL1 = OUTMOD_7;                         // CCR1 reset/set
   TA1CCR1 = velocidade;                        // CCR1 duty cycle do PWM (50%)
   TA1CTL = TASSEL_2 + MC_1;                    // SMCLK, up mode
   TA1CCTL2 = OUTMOD_7;                         // CCR1 reset/set
   TA1CCR2 = velocidade;                        // CCR1 duty cycle do PWM  (50%)
   TA1CTL = TASSEL_2 + MC_1;                  // SMCLK, up mode
   
   //CCR0 = 20000;                             // Período do PWM --> ~50Hz
   //CCTL1 = OUTMOD_7;                         // CCR1 reset/set
   //CCR1 = velocidade;                              // CCR1 duty cycle do PWM  (50%)
   //TACTL = TASSEL_2 + MC_1;                  // SMCLK, up mode
   
   _BIS_SR(GIE);
   parar();
   
   while (1)
   {
     ;
   }
}

#pragma vector = USCIAB0RX_VECTOR
__interrupt void USCI0RX_ISR (void)
{
   char aux;
   aux = UCA0RXBUF;
   switch (aux)
   {
   case '0' :  parar();
               break;
   case '1' :  anti_horario();
               break;
   case '2' :  horario();
               break;

   case '3' :  velocidade += 100;
                   if(velocidade > vel_maxima) velocidade = vel_maxima;
                   break;
   case '4' :  velocidade -= 100;
                   if(velocidade < 1) velocidade = 0;
                   break;
   }
}
