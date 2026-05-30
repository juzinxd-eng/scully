import { createFileRoute } from "@tanstack/react-router";
import { useState } from "react";

export const Route = createFileRoute("/")({
  head: () => ({
    meta: [
      { title: "Restaurante Bom Sabor - Monte sua Quentinha" },
      { name: "description", content: "Monte sua quentinha no Restaurante Bom Sabor e peça pelo WhatsApp." },
      { property: "og:title", content: "Restaurante Bom Sabor" },
      { property: "og:description", content: "Monte sua quentinha e peça pelo WhatsApp." },
    ],
  }),
  component: BomSaborCardapio,
});

const TIPOS = ["Meia Quentinha", "Quentinha Inteira"];
const ENTREGAS = ["Entrega", "Retirada"];
const ARROZ_OPCOES = ["Quero", "Não Quero"];
const FEIJAO_OPCOES = ["Feijão Vermelho", "Feijão Tropeiro", "Não Quero Feijão"];
const MACARRAO_OPCOES = ["Quero", "Não Quero"];
const PROTEINAS = [
  "Frango Empanado",
  "Peixe Empanado",
  "Parmegiana de Frango",
  "Linguiça Acebolada",
  "Carne Assada",
];
const CARNES_EXTRAS = [
  "Frango Empanado",
  "Peixe Empanado",
  "Parmegiana de Frango",
  "Carne Assada",
];
const ACOMPANHAMENTOS = ["Batata Frita", "Farofa", "Banana Frita"];
const PAGAMENTOS = ["Pix", "Dinheiro", "Cartão"];

const PRECOS: Record<string, number> = {
  "Meia Quentinha": 16,
  "Quentinha Inteira": 18,
};
const PRECO_CARNE_EXTRA = 5;

function BomSaborCardapio() {
  const whatsapp = "5522996141918";

  const [tipos, setTipos] = useState<string[]>([]);
  const [entregas, setEntregas] = useState<string[]>(["Entrega"]);
  const [arroz, setArroz] = useState<string[]>([]);
  const [feijao, setFeijao] = useState<string[]>([]);
  const [macarrao, setMacarrao] = useState<string[]>([]);
  const [proteinas, setProteinas] = useState<string[]>([]);
  const [carnesExtras, setCarnesExtras] = useState<string[]>([]);
  const [acompanhamentos, setAcompanhamentos] = useState<string[]>([]);
  const [pagamentos, setPagamentos] = useState<string[]>([]);
  const [endereco, setEndereco] = useState("");
  const [obs, setObs] = useState("");

  const ehEntrega = entregas.includes("Entrega");

  const toggleItem = (
    valor: string,
    lista: string[],
    setLista: (v: string[]) => void
  ) => {
    setLista(
      lista.includes(valor) ? lista.filter((i) => i !== valor) : [...lista, valor]
    );
  };

  const toggleAcompanhamento = (valor: string) => {
    if (acompanhamentos.includes(valor)) {
      setAcompanhamentos(acompanhamentos.filter((i) => i !== valor));
    } else if (acompanhamentos.length < 2) {
      setAcompanhamentos([...acompanhamentos, valor]);
    }
  };

  const toggleUnico = (
    valor: string,
    lista: string[],
    setLista: (v: string[]) => void
  ) => {
    setLista(lista.includes(valor) ? [] : [valor]);
  };

  const enviarWhatsapp = (mensagem: string) => {
    window.open(
      `https://wa.me/${whatsapp}?text=${encodeURIComponent(mensagem)}`,
      "_blank"
    );
  };

  const handlePedido = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    if (tipos.length === 0) {
      alert("Selecione ao menos um tipo de quentinha.");
      return;
    }

    const subtotal = tipos.reduce((acc, t) => acc + (PRECOS[t] ?? 0), 0);
    const extras = carnesExtras.length * PRECO_CARNE_EXTRA;
    const taxa = ehEntrega ? 2 : 0;
    const total = subtotal + extras + taxa;

    const enderecoTexto = ehEntrega
      ? `📍 Endereço: ${endereco}\n`
      : "📦 Pedido para retirada no local\n";
    const taxaTexto = ehEntrega ? "🚚 Taxa de entrega: R$ 2,00\n" : "";
    const extrasTexto =
      carnesExtras.length > 0
        ? `💵 Adicional de carnes: R$ ${extras
            .toFixed(2)
            .replace(".", ",")} (${carnesExtras.length} x R$ 5,00)\n`
        : "";

    const mensagem =
      `🍽️ Pedido - Restaurante Bom Sabor\n\n` +
      `📦 Tipo(s): ${tipos.join(", ")}\n` +
      `🛵 Forma: ${entregas.join(", ") || "Não informado"}\n` +
      `🍚 Arroz: ${arroz.join(", ") || "Não informado"}\n` +
      `🫘 Feijão: ${feijao.join(", ") || "Não informado"}\n` +
      `🍝 Macarrão: ${macarrao.join(", ") || "Não informado"}\n` +
      `🥩 Proteínas: ${proteinas.join(", ") || "Nenhuma"}\n` +
      `➕ Carnes adicionais: ${carnesExtras.join(", ") || "Nenhuma"}\n` +
      `🍟 Acompanhamentos: ${acompanhamentos.join(", ") || "Nenhum"}\n` +
      `💳 Pagamento: ${pagamentos.join(", ") || "Não informado"}\n` +
      enderecoTexto +
      `📝 Observação: ${obs || "Nenhuma"}\n\n` +
      extrasTexto +
      taxaTexto +
      `💰 Total: R$ ${total.toFixed(2).replace(".", ",")}`;

    enviarWhatsapp(mensagem);
  };

  const CheckGroup = ({
    titulo,
    legenda,
    opcoes,
    lista,
    setLista,
    onToggle,
    isDisabled,
    precos,
  }: {
    titulo: string;
    legenda?: string;
    opcoes: string[];
    lista: string[];
    setLista?: (v: string[]) => void;
    onToggle?: (valor: string) => void;
    isDisabled?: (valor: string) => boolean;
    precos?: Record<string, number>;
  }) => (
    <div>
      <label className="block font-semibold mb-2">{titulo}</label>
      {legenda && (
        <p className="text-sm text-gray-500 font-medium mb-2">{legenda}</p>
      )}
      <div className="space-y-2">
        {opcoes.map((op) => {
          const marcado = lista.includes(op);
          const bloqueado = !marcado && (isDisabled?.(op) ?? false);
          const preco = precos?.[op];
          return (
            <label
              key={op}
              className={`flex items-center gap-3 border rounded-xl p-3 cursor-pointer transition ${
                marcado ? "bg-green-50 border-green-400" : "bg-white"
              } ${bloqueado ? "opacity-50 cursor-not-allowed" : ""}`}
            >
              <input
                type="checkbox"
                checked={marcado}
                disabled={bloqueado}
                onChange={() =>
                  onToggle
                    ? onToggle(op)
                    : setLista && toggleItem(op, lista, setLista)
                }
                className="w-5 h-5 accent-green-600"
              />
              <div className="flex-1 flex justify-between items-center">
                <span className="font-semibold">{op}</span>
                {preco !== undefined && (
                  <span className="text-green-700 font-bold">R$ {preco.toFixed(2).replace(".", ",")}</span>
                )}
              </div>
            </label>
          );
        })}
      </div>
    </div>
  );

  return (
    <div className="min-h-screen bg-gray-100 flex justify-center p-4">
      <div className="w-full max-w-md bg-white rounded-3xl overflow-hidden shadow-2xl">
        <div className="bg-black text-white p-6 text-center">
          <h1 className="text-3xl font-bold">Restaurante Bom Sabor</h1>
          <p className="text-gray-300 mt-2">Monte sua quentinha</p>
        </div>

        <div className="p-5">
          <div className="bg-red-50 border border-red-200 rounded-2xl p-5">
            <div className="flex items-center justify-between mb-4">
              <div>
                <h2 className="text-2xl font-bold">Monte sua Quentinha</h2>
                <p className="text-gray-600">Escolha meia ou inteira</p>
              </div>
            </div>

            <form onSubmit={handlePedido} className="space-y-4">
              <CheckGroup
                titulo="Tipo de quentinha"
                legenda="Escolha apenas 1"
                opcoes={TIPOS}
                lista={tipos}
                onToggle={(v) => toggleUnico(v, tipos, setTipos)}
                precos={PRECOS}
              />
              <CheckGroup
                titulo="Entrega ou retirada"
                legenda="Escolha apenas 1"
                opcoes={ENTREGAS}
                lista={entregas}
                onToggle={(v) => toggleUnico(v, entregas, setEntregas)}
              />
              <CheckGroup
                titulo="Arroz"
                legenda="Escolha apenas 1"
                opcoes={ARROZ_OPCOES}
                lista={arroz}
                onToggle={(v) => toggleUnico(v, arroz, setArroz)}
              />
              <CheckGroup
                titulo="Tipo de Feijão"
                legenda="Escolha apenas 1"
                opcoes={FEIJAO_OPCOES}
                lista={feijao}
                onToggle={(v) => toggleUnico(v, feijao, setFeijao)}
              />
              <CheckGroup
                titulo="Macarrão"
                legenda="Escolha apenas 1"
                opcoes={MACARRAO_OPCOES}
                lista={macarrao}
                onToggle={(v) => toggleUnico(v, macarrao, setMacarrao)}
              />
              <CheckGroup
                titulo="Escolha a proteína"
                legenda="Escolha apenas 1"
                opcoes={PROTEINAS}
                lista={proteinas}
                onToggle={(v) => toggleUnico(v, proteinas, setProteinas)}
              />
              <CheckGroup
                titulo="Carnes adicionais"
                legenda="Cada carne adicional + R$ 5,00"
                opcoes={CARNES_EXTRAS}
                lista={carnesExtras}
                setLista={setCarnesExtras}
              />
              <CheckGroup
                titulo="Acompanhamentos"
                legenda={`Escolha até 2 (${acompanhamentos.length}/2)`}
                opcoes={ACOMPANHAMENTOS}
                lista={acompanhamentos}
                onToggle={toggleAcompanhamento}
                isDisabled={() => acompanhamentos.length >= 2}
              />
              <CheckGroup
                titulo="Forma de pagamento"
                legenda="Escolha apenas 1"
                opcoes={PAGAMENTOS}
                lista={pagamentos}
                onToggle={(v) => toggleUnico(v, pagamentos, setPagamentos)}
              />

              {ehEntrega && (
                <input
                  type="text"
                  value={endereco}
                  onChange={(e) => setEndereco(e.target.value)}
                  required
                  placeholder="Digite seu endereço"
                  className="w-full border rounded-xl p-3"
                />
              )}

              <textarea
                value={obs}
                onChange={(e) => setObs(e.target.value)}
                placeholder="Observação do pedido"
                className="w-full border rounded-xl p-3 h-24"
              />

              {ehEntrega && (
                <div className="bg-green-50 border border-green-200 rounded-2xl p-4">
                  <div className="flex justify-between items-center">
                    <span className="font-semibold">Taxa de entrega</span>
                    <span className="font-bold text-green-700">R$ 2,00</span>
                  </div>
                </div>
              )}

              <button
                type="submit"
                className="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-4 rounded-2xl text-lg"
              >
                Enviar Pedido no WhatsApp
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
  );
}
