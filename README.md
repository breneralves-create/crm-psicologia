import { useState, useEffect, useCallback } from "react";
import Sidebar from "@/components/dashboard/Sidebar";
import PatientsPage from "@/components/dashboard/PatientsPage";
import SessionPage from "@/components/dashboard/SessionPage";
import RecordPage from "@/components/dashboard/RecordPage";
import NewPatientModal from "@/components/dashboard/NewPatientModal";
import DeletePatientModal from "@/components/dashboard/DeletePatientModal";
import { listarPacientes } from "@/lib/api";
import type { Paciente } from "@/lib/api";
import { toast } from "sonner";

const Index = () => {
  const [page, setPage] = useState("pacientes");
  const [pacientes, setPacientes] = useState<Paciente[]>([]);
  const [loading, setLoading] = useState(true);
  const [pacienteAtivo, setPacienteAtivo] = useState<Paciente | null>(null);
  const [showNewModal, setShowNewModal] = useState(false);
  const [deleteTarget, setDeleteTarget] = useState<Paciente | null>(null);
  const [darkMode, setDarkMode] = useState(false);

  const loadPacientes = useCallback(async () => {
    setLoading(true);
    try {
      const data = await listarPacientes();
      setPacientes(data);
    } catch {
      toast.error("Erro ao carregar pacientes");
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => { loadPacientes(); }, [loadPacientes]);

  useEffect(() => {
    document.documentElement.classList.toggle("dark", darkMode);
  }, [darkMode]);

  const navigate = (p: string) => {
    if ((p === "sessao" || p === "prontuario") && !pacienteAtivo) {
      toast.error("Selecione um paciente na lista primeiro");
    }
    setPage(p);
    if (p === "pacientes") loadPacientes();
  };

  const goSessao = (p: Paciente) => {
    setPacienteAtivo(p);
    setPage("sessao");
  };

  const goProntuario = (p: Paciente) => {
    setPacienteAtivo(p);
    setPage("prontuario");
  };

  return (
    <div className="min-h-screen bg-background transition-colors duration-300">
      <Sidebar
        activePage={page}
        onNavigate={navigate}
        pacienteCount={pacientes.length}
        hasPaciente={!!pacienteAtivo}
        darkMode={darkMode}
        onToggleDark={() => setDarkMode(!darkMode)}
      />

      <main className="ml-64 min-h-screen">
        <div className="p-9 px-11">
          {page === "pacientes" && (
            <PatientsPage
              pacientes={pacientes}
              loading={loading}
              onNewPatient={() => setShowNewModal(true)}
              onSelectSessao={goSessao}
              onSelectProntuario={goProntuario}
              onDeletePatient={(p) => setDeleteTarget(p)}
            />
          )}
          {page === "sessao" && (
            <SessionPage paciente={pacienteAtivo} onGoPatients={() => navigate("pacientes")} />
          )}
          {page === "prontuario" && (
            <RecordPage paciente={pacienteAtivo} onGoPatients={() => navigate("pacientes")} />
          )}
        </div>
      </main>

      <NewPatientModal open={showNewModal} onClose={() => setShowNewModal(false)} onCreated={loadPacientes} />
      <DeletePatientModal
        paciente={deleteTarget}
        open={!!deleteTarget}
        onClose={() => setDeleteTarget(null)}
        onDeleted={() => { loadPacientes(); setPacienteAtivo(null); }}
      />
    </div>
  );
};

export default Index;

